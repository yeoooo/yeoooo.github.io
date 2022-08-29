---
title: "[Spring] Spring 트랜잭션"
excerpt: "SW Academy day 20"

categories:
    Spring
tags:
    프레임워크
    백엔드
toc: true
comments: true
use_math: true
---

<style type = 'text/css'>
    .o{
    font-weight: bold;
    color:orange;
    }
</style>


# Spring Transaction

Repository에서 활용하게 될 Transaction은 PlatformTransactionManager인터페이스를 통해 사용하게 된다.

인터페이스는 `DataSourceTransactionManager`, `JTA TransactionManager`, `HibernateTransaction` 클래스 등에 의해 구체화 되며 이를 정리해서 구상도를 그리면 다음과 같다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "transaction_concept.png" src = "../../assets/images/spring/transaction_concept.png"></p> 

트랜잭션을 직접 코드로 만들어주기 위해서는 connection을 이용해 아래와 같이 try문에서 코드를 성공적으로 실행했을 시에 commit을 시도하고 예외가 발생했을 때는 catch문의 connection.roleback()통해 롤백한다.

```java
public void transactionTest(Customer customer) {
        String updateNameSql = "UPDATE customers SET name = ? WHERE customer_id = UUID_TO_BIN(?)";
        String updateEmailSql = "UPDATE customers SET email = ? WHERE customer_id = UUID_TO_BIN(?)";

        Connection connection = null;
        try {
            connection = DriverManager.getConnection("jdbc:mysql://localhost/order_mgmt", "root", "root1234!");
            connection.setAutoCommit(false);
            try (
                    var updateNameStatement = connection.prepareStatement(updateNameSql);
                    var updateEmailStatement = connection.prepareStatement(updateEmailSql);
            ) {
                updateNameStatement.setString(1, customer.getName());
                updateNameStatement.setBytes(2, customer.getCustomerId().toString().getBytes());
                updateNameStatement.executeUpdate();

                updateEmailStatement.setString(1, customer.getEmail());
                updateEmailStatement.setBytes(2, customer.getCustomerId().toString().getBytes());
                updateEmailStatement.executeUpdate();
                connection.setAutoCommit(true);
            }
        } catch (SQLException exception) {
            if (connection != null) {
                try {
                    connection.rollback();
                    connection.close();
                } catch (SQLException throwable) {
                    logger.error("Got error while closing connection", throwable);
                    throw new RuntimeException(exception);
                }
            }
            logger.error("Got error while closing connection", exception);
            throw new RuntimeException(exception);
        }
    }
```

connection을 통한 transaction이 아닌 platform Transaction Manager를 이용해 transaction을 구현하면 아래와 같이 할 수 있다.

jdbc를 이용해 코드가 많이 줄어들었지만, Platform Transaction Manager를 통해 Connection이 아닌 트랜잭션을 따로 관리할 수 있게 되었다. 

```java
public void testTransaction(Customer customer) {
        var transaction = transactionManager.getTransaction(new DefaultTransactionDefinition());

        try {
            jdbcTemplate.update("UPDATE customers SET name = :name WHERE customer_id = UUID_TO_BIN(:customerId)", toParamMap(customer));
            jdbcTemplate.update("UPDATE customers SET email = :email WHERE customer_id = UUID_TO_BIN(:customerId)", toParamMap(customer));
            transactionManager.commit(transaction);
        } catch (DataAccessException e) {
            logger.error("Got error", e);
            transactionManager.rollback(transaction);
        }
    }
```  
<p align =  "center" style = "font-size : 12px; color:gray" >
트랜잭션을 직접 구현하게 되면 번거롭게 된다.
</p>

이 뿐 아니라 TransactionTemplate을 사용해 try/catch를 사용하지 않을 수도 있다.

TransactionTemplate은 Transaction Manager를 인자로 받기 때문에, TransactionManager의 생성자를 만들고 TransactionTemplate을 사용해야한다. 이 와중에 TransactionManager는 Datasource를 인자로 받기 때문에 약간의 번거로움이 느껴진다. 또한 이 둘 모두 Bean으로 등록해주어야 사용할 수 있다.  

```java
@Bean
public PlatformTransactionManager platformTransactionManager(DataSource dataSource) {
    return new DataSourceTransactionManager(dataSource);
}

@Bean
public TransactionTemplate transactionTemplate(PlatformTransactionManager platformTransactionManager) {
    return new TransactionTemplate(platformTransactionManager);
}
```

하지만 이를 사용할 때는 간편히 `execute(new TransactionCallback{With/withOut}Result)` 의Override할 수 있는 구현체 안에 실행하고자 하는 SQL을 넣어 사용할 수 있다.

```java
public void testTransaction(Customer customer) {
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        jdbcTemplate.update("UPDATE customers SET name = :name WHERE customer_id = UUID_TO_BIN(:customerId)", toParamMap(customer));
        jdbcTemplate.update("UPDATE customers SET email = :email WHERE customer_id = UUID_TO_BIN(:customerId)", toParamMap(customer));
      }
    });
  }
```

이렇게 Transaction API를 직접 불러와 실행시키는 Progmattic한 방법이 아닌 Declaritive(선언적) 방법을 Spring에서 제공한다. 

간단히 Transactional 하게 동작시키고 싶은 클래스나 Method에 `@Transactional` 을 선언함으로써 동작시킬 수 있다.  
```java
@Transactional
public void SomeRepository(){
    ...
}
```

## Transaction 전파

> 특정 동작중인 트랜잭션이 동작중인 과정에서 다른 트랜잭션을 실행할 경우 ‘어떻게 처리하는가' 에 대한 개념이다.

아래는 Transaction propagation의 종류이며 `@Transactional`에 `propagation` 옵션을 부여하고 사용할 수 있다.
```java  
@Transactional(propagation = Propagation.REQUIRED)
```  

tx는 트랜잭션을 의미한다.

<p align = "center"><img style = "width : 500px; height : 300px;" alt = "transaction_propagation.png" src = "../../assets/images/spring/transaction_propagation.png"></p> 

