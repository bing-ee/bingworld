## Spring으로 게시판 만들기(기본)

1. Spring Legacy Project - Spring MVC로 프로젝트 생성
   Tomcat server 연결
   

프로젝트를 최초로 생성하면 필요한 코드와 라이브러리를 다운로드하게 된다. -> .m2 폴더로 이용(.m2/repository)
* Spring Legacy project
 - 자바 1.6, 스프링 3.x가 기본이어서 수정이 필요

2. pom.xml : 생성된 프로젝트의 라이브러리 관리. 필요에 맞게 maven을 입력해 준다.
    - java를 1.8 버전으로 변경해 준다.
    - spring version을 5.x로 변경(혹은 필요 버전에 맞게 변경)
    - 'maven-compiler-plugin'의 1.6을 1.8로 수정
    - lombok 라이브러리 설치(https://projectlombok.org/download) : toString이나 getter, Setter 생성자 들을 자동으로 생성해줌
	```    
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
            <scope>provided</scope>
        </dependency>
	```
    - test를 위해 spring-test 추가
    ```
        <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>5.2.2.RELEASE</version>
		</dependency>
		```
    - junit은 4.10 이상으로 변경(4.12)

3. jdbc 연결
    - jdbc 연결을 하기 위해서는 jdbc driver 필요
    - maven에서는 지원을 하지 않기 때문에 직접 .jar파일을 연결해줌
    - java build path - external library
    - Web Deployment Assembly - java build path - ojdbc8 추가(나중에 war 파일로 만들어 질 때에도 jar 파일이 포함될 수 있도록 하는 설정)

4. jdbc 테스트 코드
    - 클래스 명 위에 @Log4j 추가
    ```
        @Log4j
        public class JDBCTest {

            static {
                try {
                    Class.forName("oracle.jdbc.driver.OracleDriver");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            
            @Test
            public void testConnection() {
                try(Connection con = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:java",
                                                                "bingworld", "bingworld")) {
                    log.info(con);
                } catch (Exception e) {
                    fail(e.getMessage());
                }
            }
        }
    ```

5. Connection Pool 설정
    - 일반적으로 여러 명의 사용자를 동시에 처리해야 하는 경우 데이터베이스를 연결할 때 connection pool을 이용
    - DataSource를 통해 매번 데이터베이스와 연결하는 방식이 아닌 미리 연결을 맺어주고 반환하는 구조를 이용 -> 성능 향상
    - HikariCP(https://github.com/brettwooldridge/HikariCP) 이용 
    - pom.xml에 추가
    ```
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>3.4.2</version>
        </dependency>
	```
    - root-context.xml 설정
         ```
            <bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
			<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"></property>
			<property name="jdbcUrl" value="jdbc:oracle:thin:@localhost:1521:java"></property>
			<property name="username" value="bingworld"></property>
			<property name="password" value="bingworld"></property>
		</bean>
		
		<!-- HikariCp configuration -->
		<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
			<constructor-arg ref="hikariConfig" />
		</bean>
        ```
    - DataSourcesTest 코드

* 스프링 프레임워크가 시작되면 먼저 스프링이 사용하는 메모리 영역을 만들게 됨 -> context
    -> 스프링에서는 applicationContext라는 이름의 객체가 만들어짐
  스프링은 자신이 객체를 생성하고 관리해야 하는 객체들에 대한 설정 필요 -> root-context(application context, 이름 무관)

6. root-context.xml 
    - namespaces - context 체크
    - <context:component-scan> 태그를 통해 컨트롤러가 들어가는 패키지 등록.
        <context:component-scan>
    
       

    
