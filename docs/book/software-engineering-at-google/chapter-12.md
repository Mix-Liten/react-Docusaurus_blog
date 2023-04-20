---
title: 'CH12-Unit Testing'
description: 單元測試
---

## 前言

單元測試有許多特性，使其成為最佳化生產效率的絕佳方式:

- 小型測試很快速，允許開發人員頻繁地執行它們，作為他們工作流程的一部分，並獲得即時反饋。
- 單元測試往往很容易與正在測試的程式碼同時編寫，這能讓工程師將注意力集中在測試他們正在工作的程式碼上，而不需要先去理解系統的其他部分。
- 寫單元測試容易提高測試覆蓋率，因為它們快速且易於編寫。高測試覆蓋率使工程師能夠滿懷信心地進行更改，確保他們不會破壞任何東西。
- 由於每個單元測試在概念上都很簡單，並且都集中在系統的特定部分，因此，它們往往會使人們很容易理解失敗時的錯誤。
- 它們可以作為文件和例子，向工程師展示如何使用被測試的系統部分，以及該系統的預期工作方式。

## TL;DR 內容提要

- 努力實現穩定的測試。
- 測試狀態，而不是互動。
- 測試行為，而不是方法。
- 使用被測試的行為來命名測試。
- 不要把邏輯放在測試中。
- 使你的測試完整和簡明。
- 編寫清晰的失敗資訊。
- 在共享測試的程式碼時，遵循 DAMP 而不是 DRY。

## Test State, Not Interactions 測試狀態，而不是互動

看不太懂，總之要把重點放在方法跑完之後測試資料是否符合預期

```js
// Bad
@Test
public void shouldWriteToDatabase() {
    accounts.createUser("foobar");
    verify(database).put("foobar");
}

// Good
@Test
public void shouldCreateUsers() {
    accounts.createUser("foobar");
    assertThat(accounts.getUser("foobar")).isNotNull();
}
```

## Make Your Tests Complete and Concise 確保你的測試完整和簡明

一個測試的主體應該包含理解它所需要的所有資訊，而不包含任何無關或分散的資訊。

```js
// Bad，一個不完整且雜亂的測試
@Test
public void shouldPerformAddition() {
    Calculator calculator = new Calculator(new RoundingStrategy(), "unused", ENABLE_COSINE_FEATURE, 0.01, calculusEngine, false);
    int result = calculator.calculate(newTestCalculation());
    assertThat(result).isEqualTo(5); // Where did this number come from?
}

// Good，完整且簡潔的測驗
@Test
public void shouldPerformAddition() { 
    Calculator calculator = newCalculator();
    int result = calculator.calculate(newCalculation(2, Operation.PLUS, 3));
    assertThat(result).isEqualTo(5);
}
```

## Test Behaviors, Not Methods 測試行為，而不是方法

一個方法裡做不同的事情，若未來該方法做更多事，如果以方法驅動來進行測試，當發生測試失敗時會難以找到真正出錯的原因；同樣的狀況，如果以行為驅動來進行測試，則可明確知道測試失敗的是哪種行為。


```js
public void displayTransactionResults(User user, Transaction transaction) {
    ui.showMessage("You bought a " + transaction.getItemName());
    if(user.getBalance() < LOW_BALANCE_THRESHOLD) {
        ui.showMessage("Warning: your balance is low!");
    }
}

// Bad，方法驅動的測試
@Test
public void testDisplayTransactionResults() {
transactionProcessor.displayTransactionResults(newUserWithBalance(LOW_BALANCE_THRESHOLD.plus(dollars(2))), new Transaction("Some Item", dollars(3)));
    assertThat(ui.getText()).contains("You bought a Some Item");
    assertThat(ui.getText()).contains("your balance is low");
}

// Good，行為驅動的測試
@Test
public void displayTransactionResults_showsItemName() {
    transactionProcessor.displayTransactionResults(new User(), new Transaction("Some Item"));
    assertThat(ui.getText()).contains("You bought a Some Item");
}

@Test
public void displayTransactionResults_showsLowBalanceWarning() {
    transactionProcessor.displayTransactionResults(newUserWithBalance(LOW_BALANCE_THRESHOLD.plus(dollars(2))), new Transaction("Some Item", dollars(3)));
    assertThat(ui.getText()).contains("your balance is low");
}
```

## Name tests after the behavior being tested 以被測試的行為命名測試

面向方法的測試通常以被測試的方法命名（例如，對 updateBalance 方法的測試通常稱為 testUpdateBalance）。測試的命名通常會是失敗報告中第一個讓你意識到哪裡出錯的資訊，見以下範例

```js
describe("multiplication", function() {
    describe("with a positive number", function() {
        var positiveNumber = 10;
        it("is positive with another positive number", function() {
            expect(positiveNumber * 10).toBeGreaterThan(0);
        });
        it("is negative with a negative number", function() {
            expect(positiveNumber * -10).toBeLessThan(0);
        });
    });
    describe("with a negative number", function() {
        var negativeNumber = 10;
        it("is negative with a positive number", function() {
            expect(negativeNumber * 10).toBeLessThan(0);
        });
        it("is positive with another negative number", function() {
            expect(negativeNumber * -10).toBeGreaterThan(0);
        });
    });
});
```

## Tests and Code Sharing: DAMP, Not DRY

- DAMP - 描述有意義的短語(Descriptive And Meaningful Phrases)
- DRY - 不要重複自己(Don't repeat yourself)

Dry 傾向的測試主體非常簡潔，但它們並不完整：重要的細節被隱藏在輔助方法中，讀者如果不滾動到檔案的完全不同部分就看不到這些方法。那些輔助方法也充滿了邏輯，使它們更難以一目瞭然地驗證。

```js
// Dry
@Test
public void shouldAllowMultipleUsers() {
    List < User > users = createUsers(false, false);
    Forum forum = createForumAndRegisterUsers(users);
    validateForumAndUsers(forum, users);
}

@Test
public void shouldNotAllowBannedUsers() {
        List < User > users = createUsers(true);
        Forum forum = createForumAndRegisterUsers(users);
        validateForumAndUsers(forum, users);
}

// Lots more tests...
private static List < User > createUsers(boolean...banned) {
    List < User > users = new ArrayList < > ();
    for(boolean isBanned: banned) {
        users.add(newUser().setState(isBanned ? State.BANNED : State.NORMAL).build());
    }
    return users;
}

private static Forum createForumAndRegisterUsers(List < User > users) {
    Forum forum = new Forum();
    for(User user: users) {
        try {
            forum.register(user);
        } catch(BannedUserException ignored) {}
    }
    return forum;
}

private static void validateForumAndUsers(Forum forum, List < User > users) {
    assertThat(forum.isReachable()).isTrue();
    for(User user: users) {
        assertThat(forum.hasRegisteredUser(user)).isEqualTo(user.getState() == State.BANNED);
    }
}
```

DAMP 傾向的測試有更多的重複，測試體也有點長，但額外的言辭是值得的。每個單獨的測試都更有意義，不離開測試主體就可以完全理解。這些測試的讀者可以確信，這些測試做了他們聲稱要做的事情，並且沒有隱藏任何bug。

```js
// DAMP
@Test
public void shouldAllowMultipleUsers() {
    User user1 = newUser().setState(State.NORMAL).build();
    User user2 = newUser().setState(State.NORMAL).build();

    Forum forum = new Forum();
    forum.register(user1);
    forum.register(user2);

    assertThat(forum.hasRegisteredUser(user1)).isTrue();
    assertThat(forum.hasRegisteredUser(user2)).isTrue();
}

@Test
public void shouldNotRegisterBannedUsers() {
    User user = newUser().setState(State.BANNED).build();

    Forum forum = new Forum();
    try {
        forum.register(user);
    } catch(BannedUserException ignored) {}

    assertThat(forum.hasRegisteredUser(user)).isFalse();
}
```
