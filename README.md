# Optional
java8 - Optional 효율적으로 사용하기

<!-- https://docs.google.com/document/d/1UxKM56um1mjGeayxmJmvALM5CDIJC17vx1-sDwoEbSs/edit# -->

## 주의사항 ##

#### 1. 리턴값으로만 쓰기를 권장한다. (메소드 매개변수 타입, 맵의 키 타입, 인스턴스 필드 타입으로 쓰지 말자.) ####
````java
// 이런식으로 사용금지X
public String getMembers(Optional<Product> s) {
	return s.get().getName();
}

getMembers(null); // 이렇게 parameter로 null이 들어올경우 에러난다. ifPresent도 마찬가지다.
// 밑에 2번과 비슷한 경우

````

#### 2. Optional을 리턴하는 메소드에서 null을 리턴하지 말자. ####
````java
// 이런식으로 사용금지X
public Optional<Product> getMembers() {
	return null;
}

// 호출했을때 product 자체가 null이므로 error 발생한다.
Optional<Product> product = getMembers();
product.ifPresent(p -> System.out.println());
````
````java
// 굳이 null을 넘겨야할때는 이렇게 해야한다. 
public Optional<Product> getMembers() {
	return Optional.empty();
}
````

#### 3. 프리미티브 타입용 Optional을 따로 있다. OptionalInt, OptionalLong,... ####
````java
// 사용금지(성능이슈) // boxing, unboxing 때문에 성능이슈 발생
Optional.of(10);
// 권장사용
OptionalInt.of(10);
````

#### 4. Collection, Map, Stream Array, Optional은 Opiontal로 감싸지 말 것 ####
````java
// 이미 null을 check하는 옵션이 있다.
````

#### 5. 예제 ####
````java
// delete의 좋은 예시
@Transactional
public void deleteById(Long id) {
	docRepository.findById(id)
	.map(p -> { 
		docRepository.delete(p);
		return p;
	})
	.orElseThrow(() -> new IdNotFoundException(id));
}
````

## API ##
1. Optional 객체 생성 3가지
	#### Optional.of() ####

	````java
	// 객체를 담고 있는 Optional 객체를 생성합니다. null이 넘어올 경우, NPE를 던지기 때문에 주의
	Optional<Member> maybeMember = Optional.of(aMember);
	```` 
	
	#### Optional.ofNullable() ####
	````java
	// null인지 아닌지 확신할 수 없는 객체를 담고 있는 Optional 객체를 생성
	Optional<Member> maybeMember = Optional.ofNullable(aMember);
	Optional<Member> maybeNotMember = Optional.ofNullable(null);
	```` 
	
	#### Optional.empty() ####
	````java
	// null을 담고 있는 Optional 객체
	Optional<Member> maybeMember = Optional.empty();
	```` 

2. Optional에 값이 있는지 없는지 확인하기
	#### isPresent() ####
	- 단지 .get() 하기위해 isPresent()를 사용한다면 .orElseThrow()를 사용하길 권장한다.
	````java
	Optional<String> opString = Optional.of("a string in optional");

	if (opString.isPresent()) {
	    System.out.println("존재합니다.");
	}
	````
	
	- isEmpty() (Java 11부터 제공)

3. Optional에 있는 값 가져오기
	- get()
	- 만약에 비어있는 Optional에서 무언가를 꺼낸다면??

4. Optional에 값이 있는 경우에 그 값을 가지고 ~를 하라.
	#### ifPresent(Consumer) ####
	````java
	Optional<Member> memberOptional = memberRepository.findById(id);
	memberOptional.ifPresent(member -> {
	    if (member.isAdmin()) {
		member.addAdminPermissions();
	    } else {
		member.addDefaultPermissions();
	    }
	});
	````

5. Optional에 값이 있으면 가져오고 없는 경우에 ~~를 리턴하라.
	- orElse(T)
	- 예) JPA로 시작하는 수업이 없다면 비어있는 수업을 리턴하라.

6. Optional에 값이 있으면 가져오고 없는 경우에 ~~를 하라.
	- orElseGet(Supplier)
	- 예) JPA로 시작하는 수업이 없다면 새로 만들어서 리턴하라.

7. **orElseThrow** 
	````java
	// Optional에 값이 없는 경우 에러를 던져라.
	sampleRepository
		.findById(seq)
		.orElseThrow(Exception::new);
	````

8. Optional에 들어있는 값 걸러내기
	- Optional filter(Predicate)

9. Optional에 들어있는 값 변환하기
	- Optional map(Function)
	````java
	// 타입을 변경할때 주로 사용되고, 타입이 변경되지 않더라도 값을 변경하거나 요소들을 가공할때 사용하기도 한다.
	Integer test = Optional.of("1").map(Integer::valueOf).orElseThrow(NoSuchElementException::new); // string to integer
	````
	
	- Optional flatMap(Function): Optional 안에 들어있는 인스턴스가 Optional인 경우에 사용하면 편리하다.

