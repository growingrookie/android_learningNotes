## MVVM模式在实际应用中的基本流程

MVVM模式将应用分为三层，各层职责如下：

- **Model层**，主要负责数据的提供。Model层提供业务逻辑的数据结构（比如，实体类），提供数据的获取（比如，从本地数据库或者远程网络获取数据），提供数据的存储。
- **View层**，主要负责界面的显示。View层不涉及任何的业务逻辑处理，它持有ViewModel层的引用，当需要进行业务逻辑处理时通知ViewModel层。
- **ViewModel层**，主要负责业务逻辑的处理。ViewModel层不涉及任何的视图操作。通过官方提供的Data Binding库，View层和ViewModel层中的数据可以实现绑定，ViewModel层中数据的变化可以自动通知View层进行更新，因此ViewModel层不需要持有View层的引用。ViewModel层可以看作是View层的数据模型和Presenter层的结合。



ViewModel层不持有View层的引用。这样进一步降低了耦合，View层代码的改变不会影响到ViewModel层。

```kotlin
    private fun initView() {
        orderItemPresenter = OrderItemPresenter(this)
        arrayObjectAdapter = ArrayObjectAdapter(orderItemPresenter)
        val itemBridgeAdapter = ItemBridgeAdapter(arrayObjectAdapter)
        my_order_grid_view?.adapter = itemBridgeAdapter
        viewModel.loadData();
    }

```

需要显示的数据通过arrayObjectAdapter绑定，当数据发生变动时Adapter会通知view

```kotlin
   private fun addObserver() {
        viewModel.orderList.observe(this) {
            if (it != null) {
                arrayObjectAdapter?.clear()
                arrayObjectAdapter?.addAll(0, it)
                arrayObjectAdapter?.notifyArrayItemRangeChanged(0, it.size)
            }
        }
    }
```

在ViewModel中会向model层请求数据并进行需要的处理

```kotlin
class MyOrderViewModel : ViewModel() {
    private val repo = GetOrderRepository()
    private var orderResult: GetOrderListResp.Data? = null
    var currentPageToken = ""
    var isFinished = false
    fun loadData() {
            orderResult = repo.getMyOrder(currentPageToken)
    }
}
```

在Repository中调用后台接口获取数据

```kotlin
class GetOrderRepository {

    suspend fun getMyOrder(pageToken: String): GetOrderListResp.Data? {
        return try {
            val result = UnifiedCgiFetcher.request(
                UnifiedCgi.GetMyOrderList,
                "pageSize" to PAGE_SIZE,
                "filter" to OrderFilter(),
                "pageToken" to pageToken
            ).fetchResult()
            (result[UnifiedCgi.GetMyOrderList] as? GetOrderListResp.Data)
        } catch (e: Exception) {
            MLog.e(TAG, "error when getMyOrder: $e")
            null
        }
    }
}
```

