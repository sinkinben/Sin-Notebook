## Qt Note

### 组件添加响应函数

本节中均使用同一个响应函数（所谓的 `slot`），代码如下：

```cpp
void MainWindow::testSlot()
{
    qDebug() << "slot got";
}
```

+ Button

  ```cpp
  QPushButton *button = new QPushButton("Push Button");
  connect(button, SIGNAL(clicked()), this, SLOT(testSlot()));
  ```

+ Menu Action

  ```cpp
  QMenu *menuFile = this->menuBar->addMenu("File");
  QAction *exitAction = menuFile->addAction("Exit");
  // the 3rd argument is lambda express
  connect(exitAction, &QAction::triggered, [=](){qDebug() << "exit";});
  ```

+ QTreeWidgetItem

  所有的 `QTreeWidgetItem` 都需要放在 `QTreeWidget` 中。

  初始化一个根节点 `rootItem`，添加若干子节点。

  ```cpp
  QTreeWidgetItem *rootItem = new QTreeWidgetItem(MainWindow::TreeTopItem);
  rootItem->setText(0, "Data Structure Type");
  
  for (int i = 0; i < MainWindow::StlTypesNumber; i++)
  {
      QTreeWidgetItem *item = new QTreeWidgetItem(MainWindow::TreeNodeItem);
      item->setText(0, stlTypes[i]);
      rootItem->addChild(item);
  }
  connect(this->leftTreeWidget, SIGNAL(itemClicked(QTreeWidgetItem*,int)), this, SLOT(clickTreeItemSlot(QTreeWidgetItem*, int)));
  ```

  `connect` 函数添加响应事件，`clickTreeItemSlot` 的参数 `QTreeWdgetItem *` 传入的是被选中的节点。最后实现这个 `private slot` 函数即可。

  ```cpp
  void MainWindow::clickTreeItemSlot(QTreeWidgetItem *item, int type)
  {
      qDebug() << "click tree item slot";
      qDebug() << item->text(0);
      qDebug() << type;
  }
  ```

  

