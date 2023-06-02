---
title: 'CH13-Test Doubles'
description: 測試替身
---

## 前言

想象一下，嘗試為一個函式編寫測試，該函式向外部伺服器傳送請求(Request)，然後將響應(Response)儲存在資料庫中。只需多付出一些努力，編寫少量的測試可能是可以做到的。但如果你需要寫成百上千個這樣的測試，你的測試套件很可能需要幾個小時才能執行，並且可能由於隨機網路故障或測試相互覆蓋資料等問題讓測試變得不穩定。

在這種情況下，測試替身就會派上用場。測試替身可以是一個物件或函式，它可以在測試中代替真正的實現，類似於特技替身可以代替電影中的演員那樣。測試替身的使用通常被稱為模擬(mocking)，但我們在本章中避免使用這個術語，因為正如我們將看到的，這個術語也被用來指代測試替身的更具體方面。

## The Impact of Test Doubles on Software Development 測試替身對軟體開發的影響

測試替身的使用給軟體開發帶來了一些複雜的問題，需要做出一些權衡：

- 可測試性(_Testability_)
為了使用測試替身，需要將程式碼庫設計成可測試的--測試應該可以用測試替身替換實際實現。例如，呼叫資料庫的程式碼需要足夠靈活，以便能夠使用測試替身來代替真正的資料庫。如果程式碼庫在設計時沒有考慮到測試，而你後來決定需要測試，那麼可能需要進行大量的提交來重構程式碼，以支援使用測試替身。

- 適用性(_Applicability_)
儘管適當地應用測試替身可以極大地提高工程速度，但其使用不當會導致測試變得脆弱、複雜且低效。當測試替身在大型程式碼庫中使用不當時，這些缺點就會被放大，這可能會導致工程師在生產效率方面的重大損失。在許多情況下，測試替身是不合適的，工程師應該傾向於使用真實的實現。

- 模擬真實度(_Fidelity_)
模擬真實度是指測試替身的行為與它所替身的真實實現的行為有多大的相似性。如果測試替身的行為與真正的實現有很大的不同，那麼使用測試替身的測試可能不會提供太多的價值——例如，想象一下，嘗試用測試替身為一個數據庫寫一個測試，這個資料庫忽略了新增到資料庫的任何資料，總是返回空結果。這樣做是完美的模擬真實度不能接受的；測試替身通常需要比實際的實現簡單得多，以便適合在測試中使用。在許多情況下，即使沒有完美的模擬真實度，使用測試替身也是合適的。使用測試替身的單元測試通常需要由執行實際實現的更大範圍的測試來支援。

## TL;DR 內容提要

- 真實實現應優先於測試替身。
- 如果在測試中不能使用真實實現，那麼偽造實現通常是理想的解決方案。
- 過度使用 Stub 會導致測試不明確和脆弱。
- 在可能的情況下，應避免互動測試：因為互動測試會暴露被測系統的實現細節，所以會導致測試不連貫。

## Basic Concepts 基本概念

使用依賴注入(_Dependency injection_)的技巧

Stubbing 意思是指在測試中 hardcode 替換測試中的真實實現；比如現在要測試手電筒，在測試手電筒的開關是否能正常運作之前，我先塞入一顆有電的電池。如果在測試之前要先假設許多的前提，這個測試本身的意義會變得薄弱，且假設的越多越偏離真實。

## Prefer State Testing Over Interaction Testing 推薦狀態測試而非互動測試

```js
// State testing
@Test
public void sortNumbers() {
    NumberSorter numberSorter = new NumberSorter(quicksort, bubbleSort);
    // Call the system under test.
    List sortedList = numberSorter.sortNumbers(newList(3, 1, 2));
    // Validate that the returned list is sorted. It doesn’t matter which
    // sorting algorithm is used, as long as the right result was returned.
    assertThat(sortedList).isEqualTo(newList(1, 2, 3));
}

// Interaction testing
@Test 
public void sortNumbers_quicksortIsUsed() {
    // Pass in test doubles that were created by a mocking framework.
    NumberSorter numberSorter = new NumberSorter(mockQuicksort, mockBubbleSort);
    // Call the system under test.
    numberSorter.sortNumbers(newList(3, 1, 2));
    // Validate that numberSorter.sortNumbers() used quicksort. The test
    // will fail if mockQuicksort.sort() is never called (e.g., if
    // mockBubbleSort is used) or if it’s called with the wrong arguments.
    verify(mockQuicksort).sort(newList(3, 1, 2));
}
```
