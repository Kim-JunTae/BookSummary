# Chapter11 CompletableFuture: 조합할 수 있는 비동기 프로그래밍

- 이 장의 내용
    - 비동기 작업을 만들고 결과 얻기
    - 비블록 동작으로 생산성 높이기
    - 비동기 API 설계와 구현
    - 동기 API를 비동기적으로 소비하기
    - 두 개 이상의 비동기 연산을 파이프라인으로 만들고 합치기
    - 비동기 작업 완료에 대응하기

- 개요
    - 소프트웨어 구현 방법에 큰 변화를 불러온 두 가지 추세
        1. 애플리케이션을 실행하는 하드웨어와 관련된 변화

            멀티코어 프로세서의 등장으로 온전히 활용할 수 있도록 소프트웨어를 구현(7장)

        2. 애플리케이션 구조 특히 애플리케이션끼리 어떻게 상호작용하는가와 관련된 변화

            모든 데이터를 자체적으로 처리하는 웹사이트나 네트워크 애플리케이션을 찾기 어렵다.
            → 즉, 다중 소스로부터 콘텐츠를 모으고 처리해서 최종 사용자에게 적절하게 제공하게 될 것이다.

            - 전형적인 매시업 애플리케이션

            ![Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled.png](Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled.png)

            멀리 있는 서비스의 응답을 기다리는 동안 우리 계산이 블록되면서 수많은 CPU 사이클을 낭비하고 싶진 않다.

            이 상황은 멀티태스크 프로그래밍의 두 가지 특징(병렬성과 동시성)을 잘 보여준다.
            7장에서 살펴본 포크/조인 프레임워크와 병렬 스트림은 훌륭한 병렬화 도구이다. 

            - 동시성과 병렬성

            ![Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled%201.png](Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled%201.png)

- Future

    # Future

    자바 5부터 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공
    비동기 계산을 모델링하는데 이용하거나 계산이 끝났을 때 결과에 접근할 수 있는 레퍼런스를 제공한다.

    예시 : 세탁소에 드라이클리닝을 맡기고 언제 끝날지 적힌 영수증(Future)을 받아 진행되는 동안 우리가 원하는 일을 할 수 있다.

    - 자바 8 이전의 예제 코드

    ```java
    //시간이 오래 걸리는 작업을 Callable 객체 내부로 감싼 다음에 ExecutorService에 제출해야 한다.

    //스레드 풀에 태스크를 제출하려면 ExecutorService를 만들어야 한다.
    ExecutorService executor = Executors.newCachedThreadPool();
    //Callable을 ExecutorService로 제출한다.
    Future<Double> future = executor.submit(new Callable<Double>() {
    	public Double call() {
    		//시간이 오래 걸리는 작업은 다른 스레드에서 비동기적으로 실행한다.
    		return doSomeLongCoputation();
    	}
    });

    //비동기 작업을 수행하는 동안 다른 작업을 한다.
    doSomethingElse();
    try{
    	//비동기 작업의 결과를 가져온다. 결과가 준비되어 있지 않으면 호출 스레드가 블록된다. 하지만 최대 1초까지만 기다린다.
    	Double result = future.get(1, TimeUnit.SECONDS);
    } catch(ExecutionException ee) {
    	//계산 중 예외 발생
    } catch(InterruptedException ie) {
    	//현재 스레드에서 대기 중 인터럽트 발생
    } catch(TimeoutException te) {
    	//Future가 완료되기 전에 타임아웃 발생
    } 
    ```

    - Future로 시간이 오래 걸리는 작업을 비동기적으로 실행하기

    ![Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled%202.png](Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled%202.png)

    ### Future 제한

    여러 Future의 결과가 있을 때 이들의 의존성은 표현하기가 어렵다.

    - '오래 걸리는 A라는 계산이 끝나면 그 결과를 다른 오래 걸리는 계산 B로 전달하시오
    그리고 B의 결과가 나오면 다른 질의의 결과와 B의 결과를 조합하시오' 와 같은 요구사항을 쉽게 구현할 수 있어야 한다.
    - 쉽지 않기 때문에 선언형 기능이 필요하다
    - CompletableFuture 클래스(Future 인터페이스를 구현한 클래스)
    Stream과 CompletableFuture는 비슷한 패턴, 즉 람다 표현식과 파이프라이닝을 활용한다.
    따라서 Future와 CompletableFuture의 관계를 Collection과 Stream의 관계에 비유할 수 있다.

    ### CompletableFuture로 비동기 애플리케이션 만들기

    - 첫째, 고객에게 비동기 API를 제공하는 방법을 배운다
    - 둘째, 동기 API를 사용해야 할 때 코드를 비블록으로 만드는 방법을 배운다.
    두 개의 비동기 동작을 파이프라인으로 만드는 방법과 두 개의 동작 결과를 하나의 비동기 계산으로 합치는 방법을 살펴본다.
    - 셋째, 비동기 동작의 완료에 대응하는 방법을 배운다.

    동기 API와 비동기 API
    - 동기 API에서는 메서드를 호출한 다음에 메서드가 계산을 완료할 때까지 기다렸다가 메서드가 반환되면 반환된 값으로 계속 다른 동작을 수행한다.(블록 호출)
    - 비동기 API에서는 메서드가 즉시 반환되며, 끝나지 못한 나머지 작업을 호출자 스레드와 동기적으로 실행될 수 있도록 다른 스레드에 할당한다.(비블록 호출)

    ## 비동기 API 구현

    ```java
    //최저가격 검색 애플리케이션
    public class Shop {
    	public double getPrice(String product) {
    		return calculatePrice(product);
    	}

    	private double calculatePrice(String product) {
    		delay();
    		return random.nextDouble() * product.charAt(0) + product.charAt(1);
    	}
    }

    //1초 지연을 흉내 내는 메서드 : 다른 외부 서비스에 접근하여 오래걸리는 작업을 대체할 delay 메서드
    public static void delay(){
    	try {
    		Thread.sleep(1000L);
    	} catch (InterruptedException e) {
    		throw new RuntimeException(e);
    	}
    }

    //사용자가 이 API(최저가격 검색 애플리케이션)을 호출하면 비동기 동작이 완료될 때까지 1초동안 블록된다.
    //위 메서드를 사용해서 네트워크 상의 모든 온라인상점의 가격을 검색해야 하므로 
    //블록동작은 바람직하지 않다. 동기 메서드를 비동기 메서드로 전환해 보자.
    ```

    - 동기 메서드를 비동기 메서드로 변환

    ```java
    //비동기 메서드 : 예제 11-4
    public Future<Double> getPriceAsync(String product) {
    	//계산 결과를 포함할 CompletableFuture를 생성	
    	CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    	new Thread( () -> {
    							//다른 스레드에서 비동기적으로 계산을 수행한다.
    							double price = calculatePrice(product);
    							//오랜 시간이 걸리는 계산이 완료가 되면 Future에 값을 설정한다.
    							futurePrice.complete(price);
    	}).start();
    	//계산 결과가 완료되길 기다리지 않고 Future를 반환한다.
    	return futurePrice;
    }

    //비동기 API 사용 : 예제 11-5
    Shop shop = new Shop("BestShop");
    long start = System.nanoTime();
    //(1) 상점에 제품가격 정보 요청
    Future<Double> futurePrice = shop.getPriceAsync("my favorite product");
    long invocationTime = ((System.nanoTime() - start) / 1_000_000);
    System.out.println("Invocation returned after " + invocationTime + " msecs");

    //제품의 가격을 계산하는 동안
    //(2) 다른 상점 검색 등 다른 작업 수행
    doSomethingElse();
    try {
    	//가격 정보가 있으면 Future에서 가격정보를 읽고, 가격 정보가 없으면 가격 정보를 받을 때까지 블록한다.
    	double price = futurePrice.get();
    	System.out.printf("Price is %.2f%n", price);
    } catch(Exception e) {
    	throw new RuntimeException(e);
    }
    long retruevalTime = ((System.nanoTime() - start) / 1_000_000);
    System.out.println("Price returned after " + retrievalTime + " msecs");

    //결과
    Invocation returned after 43 msecs
    Price is 123.26
    Price returned after 1045 msecs
    ```

    - 에러 처리 방법

    가격을 계산하는 동안 에러가 발생하면 해당 스레드에만 영향을 미친다. 즉 에러가 발생해도 가격 계산은 계속 진행되며 일의 순서가 꼬인다.

    → 클라이언트는 타임아웃값을 받는 get 메서드의 오버로드 버전을 만들어 이 문제를 해결할 수 있다. 이처럼 블록 문제가 발생할 수 있는 상황에서는 타임아웃을 활용하는 것이 좋다.

    ```java
    //문제점을 개선한 코드, CompletableFuture 내부에서 발생한 에러 전파
    public Future<Double> getPriceAsync(String product) {
    	CompletableFuture<Double> futurePrice = new CompletableFuture<>();
    	new Thread( () -> {
    							try {
    								double price = calculatePrice(product);
    								//계산이 정상적으로 종료되면 Future에 가격 정보를 저장한채로 Future를 종료한다.
    								futurePrice.complete(price);
    							} catch (Exception ex) {
    								//도중에 문제가 발생하면 발생한 에러를 포함시켜 Future를 종료한다.
    								futurePrice.completeExceptionally(ex);
    							}
    	}).start();
    	//계산 결과가 완료되길 기다리지 않고 Future를 반환한다.
    	return futurePrice;
    }
    ```

    - 팩토리 메서드 supplyAsync로 CompletableFuture 만들기

    ```java
    public Future<Double> getPriceAsync(String product) {
    	return CompletableFuture.supplyAsync(() -> calculatePrice(product));
    }
    ```

- 비블록 코드 만들기

    ```java
    //동기 API를 이용하여 최저가격 검색 애플리케이션 개발
    //상점 리스트
    List<Shop> shops = Arrays.asList(new Shop("BestPrice"),
    																 new Shop("LetsSaveBig"),
    																 new Shop("MyFavoriteShop"),
    																 new Shop("BuyItAll"));

    //제품명 입력시 상점이름과 제품가격 문자열 정보를 포함하는 List 반환 메서드
    //1. 기본
    public List<Stirng> findPrices(String product){
    	return shops.stream()
    					    .map(shop -> String.format("%s price is %.2f",
    																				 shop.getName(), shop.getPrice(product)))
    						  .collect(toList());
    }
    //1.1. 결과와 성능 확인
    long start = System.nanoTime();
    System.out.println(findPrices("myPhone27S"));
    long duration = (System.nanoTime() - start) / 1_000_000;
    System.out.println("Done in " + duration + " msecs");

    // [BestPrice price is 123.26, LetsSaveBig price is 169.47, MyFavoriteShop price is 214.13, BuyltAll price is 184.74]
    // Done in 4032 msecs

    //2. 병렬 스트림으로 요청 병렬화 하기
    public List<Stirng> findPrices(String product){
    	return shops.**parallelStream()**
    					    .map(shop -> String.format("%s price is %.2f",
    																				 shop.getName(), shop.getPrice(product)))
    						  .collect(toList());
    }

    //2.1. 결과와 성능확인
    // [BestPrice price is 123.26, LetsSaveBig price is 169.47, MyFavoriteShop price is 214.13, BuyltAll price is 184.74]
    // Done in 1180 msecs
    ```

    → 더 개선할 수는 없을까? CompletableFuture 기능을 활용해서 findPrices 메서드의 동기 호출을 비동기 호출로 바꿔보자

    ### CompletableFuture로 비동기 호출 구현하기

    ```java
    //CompletableFuture로 각각의 가격을 비동기적으로 계산한다.
    public List<Stirng> findPrices(String product){
    	List<CompletableFuture<String>> priceFutures =
    							shops.stream()
    					    .map(shop -> CompletableFuture.supplyAsync(
    														() -> shop.getName() + " price is " + 
    															    shop.getPrice(product)))
    						  .collect(Collectors.toList());

    	return priceFutures.stream()
    										 .map(CompletableFuture::join) //모든 비동기 동작이 끝나길 기다린다.
    									   .collect(toList());
    }

    //결과와 성능확인
    // [BestPrice price is 123.26, LetsSaveBig price is 169.47, MyFavoriteShop price is 214.13, BuyltAll price is 184.74]
    // Done in 2005 msecs
    ```

    - 스트림의 게으름 때문에 순차 계산이 일어나는 이유와 순차계산을 회피하는 방법

    ![Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled%203.png](Chapter11%20CompletableFuture%20%E1%84%8C%E1%85%A9%E1%84%92%E1%85%A1%E1%86%B8%E1%84%92%E1%85%A1%E1%86%AF%20%E1%84%89%E1%85%AE%20%E1%84%8B%E1%85%B5%E1%86%BB%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%87%E1%85%B5%E1%84%83%20af7129f712904cad83676f1c4247a2e5/Untitled%203.png)

    - 만족할 만한 결과가 아니다!

    ### 더 확장성이 좋은 해결 방법

    - 검색해야할 다섯 번째 상점이 추가되었을때 순차버전에서는 5025 msecs, 
    병렬에서는 2177 msecs, CompletableFuture에서는 2006 msecs
    - 9개 상점에서는?
    병렬 3143, CompletableFuture 3009
    → 결과적으로는 비슷하지만 CompletableFuture에서는 병렬 스트림버전에 비해 작업에 이용할 수 있는 다양한 Executor를 지정할 수 있다는 장점이 있다.

    ### 커스텀 Executor 사용하기

    - 스레드 풀 크기 조절

        자바 병렬 프로그래밍(Java Concurrency in Practice), 스레드 풀의 최적값을 찾는 방법 제안

        대략적인 CPU 활용 비율 계산 N_thread = N_cpu * U_cpu * (1 + W/C)
        N_cpu : Runtime.getRuntime().availableProcessors()가 반환하는 코어수
        U_cpu : 0 과 1 사이의 값을 갖는 CPU 활용 비율
        W/C는 대기시간과 계산시간의 비율

    ```java
    //1. 현재 예제 애플리케이션은 상점의 응답을 대략 99% 시간만큼 기다리므로 W/C=100
    //2. 400스레드를 갖는 풀을 만들어야 함을 의미
    //3. 상점 수보다 많은 스레드를 가지고 있어봐야 사용할 가능성이 없으므로 낭비다.
    //4. 스레드 수가 많으면 서버가 크래시될 수 있으므로 
    //   하나의 Executor에서 사용할 스레드의 최대 갯수는 100개 이하로 설정하는 것이 바람직하다.

    private final Executor executor = 
    						Executors.newFixedThreadPool(Math.min(shops.size(), 100), //상점 수 만큼의 스레드를 갖는 풀을 생성한다.(0~100)
    																				 new ThreadFactory() {
    														public Thread newThread(Runnable r) {
    															Thread t = new Thread(r);
    															t.setDaemon(true); //프로그램 종료를 방해하지 않는 데몬스레드를 사용한다.
    															return t;
    														}
    });
    ```

    - 데몬 스레드 : 주 스레드의 작업을 돕는 스레드(보조), 주 스레드가 종료되면 데몬스레드도 자동 종료된다

- 비동기 작업 파이프라인 만들기
    - 하나의 할인 서비스 제공

    ```java
    public class Discount {
    	public enum Code {
    		NONE(0), SILVER(5), GOLD(10), PLATINUM(15), DIAMOND(20);
    	
    		private final int percentage;
    	
    		Code(int percentage) {
    			this.percentage = percentage;
    		}
    	}
    	...
    }
    ```

    - 할인 서비스 제공
        - 최종 가격 계산을 위한 클래스 Quote : 가게이름, 할인 전 가격, 할인된 가격

        ...

### 요약

- 한 개 이상의 원격 외부 서비스를 사용하는 긴 동작을 실행할 때는 비동기 방식으로 애플리케이션 성능과 반응성을 향상시킬 수 있다.
- 우리 고객에게 비동기 API를 제공하는 것을 고려해야 한다. CompletableFuture의 기능을 이용하면 쉽게 비동기 API를 구현할 수 있다.
- CompletableFuture를 이용할 때 비동기 태스크에서 발생한 에러를 관리하고 전달할 수 있다.
- 동기 API를 CompletableFuture로 감싸서 비동기적으로 소비할 수 있다.
- 서로 독립적인 비동기 동작이든 아니면 하나의 비동기 동작이 다른 비동기 동작의 결과에 의존하는 상황이든 여러 비동기 동작을 조립하고 조합할 수 있다.
- CompletableFuture에 콜백을 등록해서 Future가 동작을 끝내고 결과를 생산했을 때 어떤 코드를 실행하도록 지정할 수 있다.
- CompletableFuture 리스트의 모든 값이 완료될 때까지 기다릴지 아니면 하나의 갑만 완료되길 기다릴지 선택할 수 있다.