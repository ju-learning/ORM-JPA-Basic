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

- 

# 📋 메모

- 스프링부트에서 하이버네이트를 사용하면 Entity 를 생성할때 칼럼의 필드명을 카멜케이스 → 스네이크 케이스로 자동으로 바꿔준다

- TIP) 양방향 연관관계에서 다쪽에 있는 `List` 는 `= new ArrayList<>();` 로 초기화 해주는게 Add 할때 nullpointException 이 안뜨니까 관례로 많이 씀

# 💡 팁 (단축키 등)

- 

# 🔗 레퍼런스

-
