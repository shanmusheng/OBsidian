解决 `PageView` 中嵌套的 `PageView` 在滑动时状态被清空的问题，可以使用 Flutter 的 `PageStorageKey` 来保持页面的状态。这是因为当页面滚动到外层 `PageView` 的其他页面时，Flutter 可能会回收未显示的页面，而这些页面的状态如果没有被保存就会丢失。

### 解决方案：使用 `PageStorageKey` 来保存状态

你可以通过为 `PageView` 或其子项提供一个唯一的 `PageStorageKey` 来解决这个问题。`PageStorageKey` 会将状态存储到 `PageStorage` 中，当页面被移除并重新插入时，状态会恢复。
```
PageView( key: PageStorageKey('nestedPageView'),) // 被嵌套的使用 PageStorageKey 来保存状态
```