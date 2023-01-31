# 🗂 강의 자료

[수업 자료](https://www.notion.so/273dbd3e3d7e427bb075eb4bcd2ea344)

# 🌈 강의 환경

- H2 Database

# 📝 강의 내용 정리

## **섹션 0. 강좌 소개**

### 강좌 소개

- JPA 의 정확한 매핑
- JPA 내부 동작 방식 이해

### 수업 자료

- (수업자료 다운로드)

## 섹션 1. JPA 소개

### SQL 중심적인 개발의 문제점

- 개발할때는 객체지향 방법론을 사용, 데이터베이스는 관계형 DB 를 사용 → CRUD SQL 을 작성하는 방법론들이 지루하게 반복되고 지루함 (개발자가 SQL 매퍼 역할)
- 객체와 관계형 데이터베이스는 비슷한것같지만 패러다임 자체가 다름
- 객체는 참조를 사용하고, 테이블은 왜래키를 사용한다 (`getTeam()` vs `JOIN`)
- 객체다운 모델링(참조변수)은 SQL 을 통한 INSERT 문을 작성하기 까다로움
- 객체는 자유롭게 객체 그래프를 참조할 수 있어야하는데, SQL 은 조회되지 않으면(JOIN 하지않으면) 탐색할 수 없다
    - 계층형 아키텍쳐 에서는 넘어온 데이터를 믿고 쓸 수 있어야 하는데, 탐색이 안되면 해당 엔티티에 대한 신뢰가 없어짐
- 객체지향적으로 설계할수록 코드만 지저분해짐 → 자바 컬렉션처럼 DB를 사용할 수 없을까? 하는 고민들 → JPA 가 나옴

### JPA 소개

- JPA → Java Persistence API (자바진영의 ORM 표준)
- ORM → Object Relational Mapping (객체와 관계형을 매핑)
- 자바 컬렉션을 사용하듯 객체를 사용하면 JPA 가 여러 문제점들(매핑, 패러다임불일치) 해결해준다
- 예전에 너무 막장이었던 Enterprise JavaBeans; EJB 를 잘 만들기위해 고민하던 개발자 게빈킹이 하이버네이트를 만들어버림
- 우리는? → “JPA 표준 인터페이스에서 하이버네이트 구현체를 쓰는 중”
- Entity = JPA 가 관리하는 객체
- JPA 의 성능 최적화
    - 1차 캐시와 동일성 보장
    - 트랜잭션을 지원하는 쓰기 지연
    - 지연로딩
- JPA 는 ORM 이고 ORM 은 객체와 관계형DB의 매핑을 도와주는것이기 때문에 결국 잘 사용하려면 객체지향과 RDB를 다 잘 다루어야 함

## **섹션 2. JPA 시작하기**

### Hello JPA - 프로젝트 생성

- (H2 데이터베이스 설치)
    - 최초 접속시엔 `jdbc:h2:~/test` 로 접속, 이후에는 `jdbc:h2:tcp://localhost/~/test` 로 사용
- 실무에선 거의 Gradle 을 사용하지만, 실습환경에선 Maven 을 사용
    - 라이브러리에 사용할 hibernate 버전 확인 → [https://hibernate.org/orm/](https://hibernate.org/orm/)
    - 내게 맞는 버전 찾는법: [https://spring.io/projects/spring-boot#learn](https://spring.io/projects/spring-boot#learn) → Reference Doc. → Dependency Versions
- `hibernate.dialect` 설정을 통해 JPA 는 각각의 데이터베이스만의 특정한 “방언”을 맞춰줄 수 있음

### Hello JPA - 애플리케이션 개발

- (만든 Maven 프로젝트에서 JPA를 사용해보는 `JpaMain` 클래스 예제)

```java
public class JpaMain {
	public static void main(String[] args) {
		EntityManagerFactory emf = persistence.createEntityManagerFactory("hello");

		EntityManager em = emf.createEntityManager();

		EntityTransaction tx = em.getTransaction();

		tx.begin(); // 트랜잭션 시작
		try {
			Member findMamber = em.find(Member.class, 1L);
			findMember.setName("HelloJPA");

			tx.commit(); // 트랜잭션 커밋
		} catch (Exception e) { // 에러시
			tx.rollback(); // 트랜잭션 롤백
		} finally { em.close(); }

		emf.close();
	}
}
```

- JPA 의 모든 변경은 트랜잭션 안에서 실행해야 한다.
- 엔티티 매니저는 쓰레드간에 공유 X (사용하고 버려야함)
- [Java Persistence Query Language (JPQL)](https://en.wikibooks.org/wiki/Java_Persistence/JPQL) is the query language defined by JPA. → 객체지향 SQL
    - JPQL 은 테이블이 아닌 객체(Entity)를 대상으로 쿼리를 실행함
    - 실제 테이블에 쿼리를 날려버리면 JPA 의 구조 자체가 무너지므로, 복잡한 find 쿼리같은건 객체 지향 쿼리인 JPQL 을 많이 사용

## **섹션 3. 영속성 관리 - 내부 동작 방식**

### 영속성 컨텍스트 1

- JPA 에서 가장 중요한 2가지를 뽑으라면?
    1. 객체와 관계형 데이터베이스 매핑 (설계)
    2. 영속성 컨텍스트: 실제 데이터베이스가 내부적으로 어떻게 동작하는지, “엔티티를 영구 저장하는 환경”
- 영속성 컨텍스트
    - 영속성 컨텍스트는 논리적인 개념임 (물리x)
    - 눈에 보이지 않으며, EntityManager 를 통해 영속성컨텍스트를 접근
- 엔티티의 생명주기
    - 비영속(new/transient): 객체를 생성만 한 상태
    - 영속(managed): em(EntityManager) 를 가져와서 `em.persist( .. );` 를 해서 영속화 한 상태. 영속상태가 된다고 쿼리가 날아가는게 아님
    - 준영속
    - 삭제

### 영속성 컨텍스트 2

- 영속성 컨텍스트는 내부에 1차 캐시를 들고있음
- 영속 엔티티의 동일성을 보장해줌
- 트랜잭션을 지원하는 쓰기 지연 → 모았다가 한번에 쿼리할 수 있는 기능 (배치가 아니고 실시간에선 크게 이점은 없지만 유리함)
- 엔티티 변경 감지 (더티체킹) → 커밋을 하면 flush 를 호출하는데, 이때 기존의 1차캐시 데이터와 스냅샷을 비교하고 다르면 update query를 작성해줌

### 플러시

- 플러시: 영속성 컨텍스트의 변경내용을 데이터베이스에 반영 (동기화) 하는 작업
- 실제 코드에서 커밋전에 강제로 플러시를 호출하려면 `em.flush();` 를 호출하면 됨
- JPQL 을 사용시 JPQL 은 쿼리를 직접 생성해서 호출하기때문에, 쿼리가 나가기전 자동으로 플러시를 발생한다

### 준영속 상태

- 영속상태가 되기위해선 `em.persist` 나 `em.find` 를 호출하면 된다 (호출하면 1차캐시에 올라가기때문)
- 영속상태의 객체를 `em.detach( .. )` 를 사용하면 준영속 상태로 영속상태가 해제된다

### 정리

- 영속성 컨텍스트에 대해 알아봄
- JPA 에서 매핑과 영속성컨텍스트가 매우 중요
- 엔티티에는 생명 주기가 있음

![https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0aed5f68-1cb7-4253-bad8-304669fb5f41/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230127%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230127T090454Z&X-Amz-Expires=86400&X-Amz-Signature=1584ee1cb85ebea94a9011966daf88132e568b6110674c6bca2cb003011d3102&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/0aed5f68-1cb7-4253-bad8-304669fb5f41/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230127%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230127T090454Z&X-Amz-Expires=86400&X-Amz-Signature=1584ee1cb85ebea94a9011966daf88132e568b6110674c6bca2cb003011d3102&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

- 플러시는 영속성컨텍스트를 비우지않음 (그냥 동기화임)
- 트랜잭션이라는 작업 단위가 매우 중요

## **섹션 4. 엔티티 매핑**

### 객체와 테이블 매핑

- 객체와 테이블매핑, 스키마 자동생성, 필드와 컬럼, 기본키, 연관관계 매핑에 대해 배움
- 객체와 테이블 매핑 - `@Entity`
    - 이 어노테이션이 붙으면 JPA 가 관리하는 엔티티임
    - 기본 생성자 필수

### 데이터베이스 스키마 자동 생성

- 데이터베이스의 매핑정보만 보면 어떤 쿼리를 만들어야 할지 알 수 있으므로, 로딩시점에 테이블 쿼리를 만들어줌
- 데이터베이스 방언을 활용하여 특정 DB엔진에 맞는 DDL 문을 설정해줌
- 이렇게 생성된 DDL 은 반드시 개발 환경에서만 사용

### 필드와 컬럼 매핑

- 필드의 `@Column` 속성중 `unique` 값은 랜덤값으로 배정되기때문에 보통 `@Table` 의 `uniqueConstraints` 를 많이 사용
- `@Enumerated` 를 사용할때 타입은 반드시 `ORDINAL` 이 아닌, `STRING` 을 사용 → 순서에 의존적이게 된다
- 사실상 `@Temporal` 은 옛날 자바의 `Date`, `Time` 클래스를 사용할때만 필요함

### 기본 키 매핑

- 간단히 기본키 매핑은 `@Id` 어노테이션을 사용함 → 직접 ID 를 셋팅
- 또한, `@GeneratedValue` 로 자동생성 할 수 있음
    - `IDENTITY`: 보통 MySQL 에서 사용(MySQL 의 mysql auto_increment)디비에게 자동생성을 완전 위임
    - `SEQUENCE`: 보통 오라클DB 에서 많이 사용하며, 시퀀스테이블을 사용하여 넘버링
- 권장하는 식별자 전략:
    - 키본키 제약 조건: null이면 안됨, 유일해야함, 변하면 안됨
    - 권장: Long 형 + 대체키(UUID 같은) + 키 생성전략 사용
- IDNTITY 전략의 특징
    - 데이터베이스에 실제로 나가봐야 키값을 알 수 있기때문에, 영속성컨텍스트의 1차캐시에 저장하기 위해 `em.persist` 를 호출하면 즉시 `insert` 쿼리가 나감
    - 따라서 모아서 (버퍼링해서) 한번에 쿼리가 나가는 전략을 사용할 수 없다 → 김영한님 피셜: 근데 뭐 큰 의미는 없음
- SEQUENCE 전략의 특징
    - 시퀀스 전략이면 id 값을 가져오기 위해 `em.persist` 를 호출하기 위해 다음 시퀀스를 위한 SELECT(call next value) 가 호출되고 영속성 컨텍스트에 저장됨
    - 이럴때 성능 최적화를 위해 `allocationSize` 를 사용해서 미리 시퀀스값을 미리 가져와서 사용하는 방법 → 동시성 문제도 없음

### 실전 예제 1 - 요구사항 분석과 기본 매핑

- (조금더 복잡한 예제를 통해 어떻게 설계하는지 알아보자)
- (라이브코딩으로 엔티티 설계대로 간단한 주문 Entity 들 설계)
- 데이터 중심으로 설계하면 `Member` 를 직접 호출하지 않고 `memberId` 를 가지고 찾아야하기 때문에 객체지향 스럽지 않다 → 다음시간에 연관관계 매핑을 배워보자

## **섹션 5. 연관관계 매핑 기초**

### 단방향 연관관계

- 단순히 id 를 가지고 호출하는것이 아닌, 객체 자체를 매핑해서 가져오는 방법
- (연관관계의 주인 개념이 어려움)
- 객체를 테이블에 맞춰서 `team_id` 를 직접 맵핑했을때의 문제점
    - 팀을 저장하고 회원을 저장할 때, `teamId` 를 가지고 팀을 가지고 오는게 객체지향스럽지않음
    - `team_id` 만 가지고 팀을 알 수 없으니, Member 에서 Team 을 사용하려면 계속 `em.find` 를 호출해야 함
    - 테이블은 외래키로 조인 vs 객체는 참조를 사용 → 모델링의 협력 관계를 만들 수 없음
- 단방향 연관관계는 `@ManyToOne` 과 `@JoinColumn` 을 사용하여 간단하게 가능

### 방향 연관관계와 연관관계의 주인 1- 기본

- (JPA 계의 포인터;;) 영속성컨텍스트의 메커니즘과 양방향연관관계와 연관관계 주인이 제일 어려움
- 테이블의 연관관계는 사실상 Foreign Key 하나만 가지고 양방향을 다 다룰 수 있음 but 객체는 아님
- `@OneToMany` 에서는 `mappedBy` 를 사용하여 어디에 매핑이 되어있는지 기재해준다
- 연관관계의 주인과 mappedBy
    - 객체에서의 양방향 연관관계는 사실상 단방향 연관관계가 2개 있는거임
    - 객체의 양방향에서는 어느 단방향쪽에서 업데이트를 해줘야 할지 정해야 한다 (테이블은 FK 하나로 되지만) → 둘중 하나로 주인을 정해서 관리를 해야함 = 연관관계 주인
    - 연관관계 주인에서 주인은 CRUD 가 가능, 주인이 아니면 읽기만 가능, 주인은 `mappedBy` 를 사용하지 않음, 주인이 아니면 `mappedBy` 로 주인이 누군지 써야함
    - 누구를 주인으로 해야하지? → mapped ‘By’ 니까 `mappedBy` 가 없는게 주인임, `mappedBy` 는 읽기만 가능하고 값을 넣어봐야 의미없음
    - (강사 추천) 외래키가 있는곳을 주인으로 정하는게 좋음
        - 이걸 반대로 하면 Team 을 업데이트 했는데 Member 쿼리가 나갈 수 있음
        - 성능이슈도 있음 → 뒤에서 설명
        - 1:N 에서 N쪽이 무조껀 연관관계 주인이 되는게 제일 깔끔함

### 양방향 연관관계와 연관관계의 주인 2 - 주의점, 정리

- `mappedBy` 가 붙어있으면 읽기전용이라 JPA 가 insert 할때 아무런 동작을 안씀
- 양방향 매핑시에 연관관계 주인에 반드시 값을 넣어주지 않으면 `null` 이 들어가므로 주의
- 그냥 양방향 매핑을 할때는 양쪽에 다 값을 넣어주는게 맞음
    - JPA 를 통해 불러오기 전에 `em.flush(), em.clear()` 를 해주면 문제가 안되지만, 그전에 호출해버리면 DB 에서 select 가 안나가서 데이터가 없다고 잘못 나옴
    - JPA 입장에서 보면 주인에만 값을 넣어주면 되지만, 객체지향적으로 생각하면 둘다 데이터가 있는게 맞기때문에
- 이럴때를 위해 연관관계 편의 메서드를 작성해주는게 좋다. 더 자세한 편의메서드 작성법은 책을 참고

```java
public void changeTeam(Team team) {
	this.team = team;
	team.getMembers().add(this);
}
```

- member 를 기준으로 team 을 넣을지, team 을 기준으로 member 를 넣을지 편의메서드를 작성할곳을 정해주면 됨 (한쪽에만 작성해주는게 좋음) → 어플리케이션 작성하면서 어디 있으면 되겠다 싶은곳에 넣으면됨
- 양방향 매핑시에 무한루프를 주의 (toString, lombok, JSON 생성 라이브러리 사용시)
    - Lombok 에서 toString 은 지양해라
    - Entity를 JSON 생성할때 쭉 뽑으면서 에러가 남 (Entity를 컨트롤러에서 직접 반환하는 경우) → 컨트롤러에는 그냥 Entity 를 쓰지말고, DTO 로 변환해서 반환해라
- 정리: 단방향 매핑만으로도 이미 연관관계는 매핑이 완료된거임
    - 객체입장에서 “양방향매핑”은 단점이 많아짐
    - 처음에 설계할 때, 먼저 단방향 매핑으로 싹다 설계를 완료해야함
    - (양방향매핑은 그냥 읽기 권한만 추가되는거라서) 필요시에만 양방향을 추가함 → 양방으로 바꾸는건 그냥 List 만 하나 추가해주면 되기때문
- 중요) 연관관계의 주인은 외래키의 위치를 기준으로 정해야함

### 실전 예제 2 - 연관관계 매핑 시작

- (그전엔 DB 설계대로 만들었는데, 이제 객체지향 스럽게 연관관계를 설계해보자)
- (섹션4 마지막에 했던 실전예제 1을 객체연관관계 매핑으로 변경하는 내용)
- 가급적이면 단방향 매핑이 좋은거임 → 실제 개발하다가 필요할때만 양방향 매핑
- “특정 회원의 전체 주문목록” 을 가져오는 기능을 구현하려면 “주문” 에서 특정 멤버의 정보를 가져와야지, 특정 멤버 → 멤버의 주문 으로 가는건 관심사를 끊어내지못하고 더 어렵게 설계한거라고 생각함 (주문이 필요하면 그냥 주문에서부터 시작하면됨)
- 실전에서는 JPQL 을 작성하는 경우가 꽤 많은데 이럴경우 양방향 연관관계가 필요한 경우가 많다

## 섹션 6. 다양한 연관관계 매핑

### 다대일 [N:1]

- 기본적인 연관관계 고려사항을 따져볼것이고, 여러 매핑시 고려사항을 알아봄
- 연관관계 매핑시 고려사항 3가지:
    - 다중성 → n:1, 1:n, 1:1, n:m (다대다는 실무에선 지양하는게 좋음)
    - 단(양)방향 → 테이블은 외래키 하나로 양쪽 조인으로 데이터를 가져오기때문에 “방향”이라는 개념이 없지만, 객체는 참조형 필드가 있어야해서 단/양방향이 있음. 그리고 사실 “양방향”은 아니고 단방향 2개임
    - 연관관계의 주인
- 다대일 단방향: 당연히 테이블에는 다(N) 쪽에 외래키(FK)가 가야하고, 객체는 외래키 있는쪽에 참조객체를 넣으면 됨 (가장 많이 사용하는 연관관계)
    - 여기서 양방향을 하려면 간단하게 반대쪽에 List 만 넣고 `mappedBy` 해주면 해결

### 일대다 [1:N]

![Untitled](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5859b315-f6b4-45df-8073-9f247df06d95/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230131%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230131T131925Z&X-Amz-Expires=86400&X-Amz-Signature=0ec2503d76e531091e34c471db51b27b4ece34bb8515454101a431e16141d42d&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

- 여기선 1쪽에서 외래키를 관리 (강사님은 권장하지 않는 모델임, 스펙상 스프링이 지원만 할 뿐 실무에서 지양함)
- `@OneToMany` 를 사용한 `List<Mamber> members` 한곳에 `@JoinColumn( .. )` 을 선언해주면 실제로 사용 가능
- 이렇게 매핑할 시 하이버네이트가 옆 테이블 (FK 가 있는 테이블) 의 업데이트 쿼리를 만들어줌
- 가장 큰 단점: 도메인 입장에서 Team Entity 만 수정했는데도, Member Table 의 update 가 나가기때문에 트래킹이 굉장히 힘들거나 사용이 난해해짐
- 이런 구조가 너무 필요하다면 그냥 원래대로 n:1 로 설계하고, 양방향 매핑으로 가져오는걸 권장
- 주의) `@JoinColumn( .. )` 을 안써버리면 테이블과 테이블 사이에 JPA 가 매핑테이블을 만들어서 관리함 → 테이블이 하나 더들어가니까 성능이슈도 있고, 운영에서 너무 복잡해짐
- 결론) 조금 객체지향적으로 참조를 하나 더 넣더라도 “다대일 양방향 매핑”을 사용하는것을 권장
- 일대다 양방향
    - 이런 매핑은 공식적으론 없는거지만, `@ManyToOne` 와 `@JoinColumn(name="..", insertable=false, updatable=false` 를 사용하면 insert 와 update 를 사용하지 않는 읽기전용 (==mappedBy랑 비슷한) 역할을 할 수 있음

### 일대일 [1:1]

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/80d91aa4-23b1-4203-bab4-ef3c829a42c4/Untitled.png)

- 일대일은 대칭관계이기때문에, 주테이블이나 대상테이블에 아무곳에나 외래키 선택이 가능
- 외래키 데이터베이스에 유니크제약조건을 추가해주는게 좋다
- `XXXToOne` 은 사용법이 비슷함
- (위 그림대로 설계하면 굉장히 쉽고 편리함)
- 일대일인데 대상테이블(locker) 에 FK 가 존재하는 단방향은 JPA 에서 아예 지원하지 않음
- 일대일인데 대상테이블(locker) 에 FK 가 존재하는 양방향은 그냥 위 그림에서 locker 쪽에 외래키를 넣고, 반대쪽에 읽기전용 mappedBy 를 넣어주면됨
- 외래키는 Member 에 넣는게 좋을까 Locker 에 넣는게 좋을까? (일단 정답은 없지만)
    - 만약에 1:1이 1:n 으로 바뀔 미래를 생각하면 → Locker 쪽에 FK 가 있다면 그냥 유니크 제약조건만 빼버리면됨
    - 실제 ORM 을 사용해서 개발을할 개발자 입장에선 Member 에 Locker_Id 를 FK 로 가지고있으면 비즈니스 로직에서 멤버는 대부분 가져올때 이미 FK 값이 있으므로 JOIN 을 사용하여 쉽게 Locker 의 상태를 알 수 있음. 단, Locker 가 null 일수도있음
    - 강사님은 그냥 Member 에서 Locker 를 가져가는걸 선호함 (위의 그림 그대로)
    - Locker 에 Member_Id FK 가 걸려있으면 무조껀 양방향으로 걸려있어야됨 (1:1 단방향 반대쪽 FK는 지원을 안하니까)
    - 주테이블에 외래키 → 주테이블만 조회해도 이미 Locker 정보를 알기 쉽지만, null이 들어갈 수 있음
    - 대상 테이블에 외래키 → 나중에 Locker 가 N 으로 변경되어도 수정할게 적음, 단 Entity 구조 자체가 반드시 양방향으로 해야하고 지연로딩을 사용할 수 없음

### 다대다 [N:M]

- (강사님은 실무에서 쓰면 안된다고 생각)
- 관계형 데이터베이스는 정규화된 테이블2개로 다대다 관계를 표현할 수 없음 → 따라서 연결테이블(매핑테이블) 을 사용해서 1:n 두개로 만들어야함
- 근데 객체는 N:M 구조가 그냥 가능함 (컬렉션 2개 넣으면 되니까) `@ManyToMany` 를 사용하고 동일하게 단/양방향 모두 가능
- 실무 테이블에선 연결만 하고 끝나지 않고 복잡한데, 매핑테이블에 다른 필드(칼럼)들이 필요한데 이런것들 추가가 불가능하고, 중간 테이블이 숨겨져있어서 쿼리가 굉장히 복잡하게 나감
- n:m 이 필요한경우 중간에 수동으로 매핑테이블을 만들고 1:n 두개를 만들어서 연결
- 중간 매핑테이블에서 실제로 사용할땐 PK는 의미없는 값을 넣고, FK 로 그냥 매핑하는게 대부분 괜찮았다

### 실전 예제 3 - 다양한 연관관계 매핑

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22bf2a4a-6254-4170-a451-b060d1b66315/Untitled.png)

- (1:1 관계와 N:N 관계를 추가하고 Entity 를 설계하는거 실습) `@ManyToMany` 지양하라고 했지만 보여줄려고 그냥 사용함
- (`@JoinColumn` `@ManyToOne` 등의 옵션들 한번 찾아볼 것)
    - 다:1 은 `mappedBy` 자체가 없음 (연관관계 주인이 되어야해서)

# 📋 메모

- 스프링부트에서 하이버네이트를 사용하면 Entity 를 생성할때 칼럼의 필드명을 카멜케이스 → 스네이크 케이스로 자동으로 바꿔준다
- TIP) 양방향 연관관계에서 다쪽에 있는 `List` 는 `= new ArrayList<>();` 로 초기화 해주는게 Add 할때 nullpointException 이 안뜨니까 관례로 많이 씀
- 실무에선 세터를 잘 안쓰고, 생성자에서 만들거나 빌더패턴을 사용

# 💡 팁 (단축키 등)

- 

# 🔗 레퍼런스

-
