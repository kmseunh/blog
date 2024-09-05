+++
title = "서비스 간 결합도를 줄여보자"
date = "2024-09-05"
description = "프로젝트 설계 시 독립성과 협력의 개념을 코드 구현에 어떻게 반영할 수 있는지.."
tags = [
    "2024",
    "architecture",
    "spring"
]
categories = [
    "indiv"
]
series = ["Study"]
+++

간단한 은행 입출금 프로젝트를 진행하면서 독립성과 협력의 개념에 대해 깊이 고민해 보았다. <br>
두 개념은 시스템 설계에서 각각 중요한 역할을 하지만, 때로는 서로 상충할 수도 있음을 알게 되었다.

이번 포스팅에서는 이러한 개념들이 실제 코드 구현에 어떻게 영향을 미치는지 알아보려 한다.

<p align="center"><img src="https://github.com/user-attachments/assets/b5429aaa-58d7-4538-9536-2ec53b71b81b" width="450"></p>

<!--more-->

## 독립성 vs 협력

### 독립성

독립성은 시스템이나 구성 요소가 다른 부분에 의존하지 않고 독립적으로 작동할 수 있는 특성을 의미한다. <br>
이 원칙을 따르면 각 구성 요소가 자신의 책임만을 다하고 다른 부분에 영향을 미치지 않도록 설계할 수 있다.

#### 장점

- 유지 보수성 향상: 각 구성 요소가 독립적으로 설계되면, 특정 기능에 대한 변경이 다른 기능에 미치는 영향을 최소화할 수 있다.
- 테스트 용이성: 독립적인 구성 요소는 개별적으로 테스트하기가 용이하여, 전체 시스템의 신뢰성을 높일 수 있다.

### 협력

협력은 독립적인 구성 요소들이 상호작용하여 하나의 목표를 달성하는 과정을 의미한다. <br> 독립적으로 설계된 각 요소가 협력할 때는 역할과 책임이 명확히 정의되어야 하며, 효율적인 상호작용을 통해 시스템의 전체적인 성능을 향상시킬 수 있다.

#### 장점

- 통합된 기능 구현: 개별적으로 독립적인 요소들이 협력하여 통합된 기능을 구현할 수 있다.
- 효율적인 상호작용: 역할과 책임이 명확하게 정의된 협력을 통해 시스템의 성능을 극대화할 수 있다.

## 이슈 상황

프로젝트에서 Account는 계좌 조회 및 입출금을 담당하며, Transaction은 입출금 시 트랜잭션을 기록하는 역할을 맡고 있다. <br>
기능을 명확히 분리하기 위해 DAO, Service, Controller 계층을 설계하여 각각의 독립성을 유지하고자 했다.

 • Spring Framework에서 계층을 구분하는 이유는 책임을 분리하여 코드의 가독성과 유지 보수성을 높이기 위함이다.

처음 설계할 때는 Account와 Transaction 서비스를 완전히 독립적으로 유지하고자 했다. <br>
Account 서비스는 입출금만 담당하고, Transaction 서비스는 계좌와 상관없이 트랜잭션 기록만을 처리하도록 하여, 각각의 기능이 서로 간섭하지 않도록 설계했다.

그러나 입출금을 처리하면서 계좌의 잔액 변경과 동시에 해당 내역을 기록해야 하는 요구사항이 생기자, 두 서비스 간 협력이 필수적이라는 사실을 깨달았다.

### 초기 코드: 독립성을 유지하려는 접근

입금 시 트랜잭션을 기록하는 초기 코드는 다음과 같다.

```java
@Override
@Transactional
public boolean deposit(int accountId, double amount) {
    AccountDTO account = accountDAO.getAccountById(accountId);
    if (account == null) {
        return false;
    }

    account.setBalance(account.getBalance() + amount);
    accountDAO.updateBalance(account);

    transactionService.saveTransaction(accountId, "DEPOSIT", amount);

    return true;
}
```

이 코드에서 `transactionService.saveTransaction(accountId, "DEPOSIT", amount);` 호출 부분은 서비스 간의 독립성을 유지하려는 의도로 구현되었다. <br>

- accountId, amount, transactionType과 같은 파라미터를 직접 전달함으로써 TransactionService가 특정 데이터 구조를 알지 못하도록 하려는 목적

그러나 이 방식은 직접적으로 파라미터를 받아 처리하므로, 오히려 서비스 간의 결합도를 높아지고 TransactionService의 구현 변경이 deposit 메서드에 영향을 미칠 수 있다. <br>
또한, 트랜잭션 저장 방식이나 추가적인 정보가 필요할 경우, deposit 메서드까지 수정해야 할 수 있다.

### 개선한 코드

개선된 코드에서는 TransactionDTO를 사용하여 트랜잭션 정보를 캡슐화하고 서비스 간의 결합도를 줄였다.

```java
// 개선된 코드
@Override
@Transactional
public boolean deposit(int accountId, double amount) {
    AccountDTO account = accountDAO.getAccountById(accountId);
    if (account == null) {
        return false;
    }

    account.setBalance(account.getBalance() + amount);
    accountDAO.updateBalance(account);

    TransactionDTO transaction = new TransactionDTO();
    transaction.setAccountId(accountId);
    transaction.setTransactionType("DEPOSIT");
    transaction.setAmount(amount);
    transactionService.saveTransaction(transaction);

    return true;
}
```

개선된 코드에서는 TransactionDTO를 사용하여 트랜잭션 정보를 캡슐화함으로써 데이터의 구조와 전송 방식을 명확히 했다. <br>
이로 인해 AccountService는 DTO만 제공하면 되므로 서비스 간의 결합도가 줄어든다.

DTO를 사용하면 트랜잭션 저장 방식이나 추가적인 필드가 필요할 경우, DTO만 수정하면 되며, deposit 메서드는 변경 없이 DTO를 통해 데이터를 전달할 수 있다.

<hr>

지금까지는 독립적으로 역할을 분리하는 것에 중점을 두어 왔지만, 협력의 중요성을 깨닫고 나서는, 독립성을 유지하면서도 효과적으로 협력할 수 있는 방법을 찾는 것이 얼마나 중요한지 배웠다. <br>
특히, DTO를 활용한 데이터 캡슐화가 서비스 간의 결합도를 줄이고, 코드의 가독성과 유지 보수성을 높이는 데 어떻게 기여하는지를 알게 되었다.
