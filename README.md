#UITableView支持不同类型的Cell#

在某些业务场景下，同一个UITableView需要支持多种UITableViewCell。考虑一下即时通信的聊天对话列表，不同的消息类型对应于不同的UITableViewCell，同一个tableview往往需要支持多达10种以上的cell类型，例如文本、图片、位置、红包等等。一般情况下，UITableViewCell往往会和业务数据模型绑定，来展示数据。根据不同的业务数据，对应不同的cell。本文将探讨如何有效的管理和加载多种类型的cell。

为了方便讨论，假设我们要实现一个员工管理系统。一个员工包含名字和头像。如果员工只有名字，则只展示名字，如果只有头像，则只展示头像，如果既有名字，又有头像，则需要既展示头像又展示名字。

我们用一个Person类表示员工，用三种不同的cell来处理不同的展示情况。

```
@interface Person : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) NSString *avatar;
// 创建Cell类型
@property (nonatomic, copy) NSString * cellClassName;
// 返回tableViewCell
- (UITableViewCell *)createdTableViewCell:(UITableView *)tableView;

@end
```

```
/*负责展示只有名字的员工*/

@interface TextCell : BaseCell

- (void)setPerson:(Person *)p;

@end

```

```
/*负责展示只有头像的员工*/

@interface ImageCell : BaseCell

- (void)setPerson:(Person *)p;

@end

```

```
/*负责展示只有既有名字又有头像的员工*/

@interface TextImageCell : BaseCell

- (void)setPerson:(Person *)p;

@end

```
这三个类都继承了BaseCell，BaseCell继承UITableViewCell

```
@interface BaseCell : UITableViewCell

- (void)setPerson:(Person *)p;

@end

```
下面我们在UITableView的delegate来处理展示Cell
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    Person *p = _persons[indexPath.row];
    return [p createdTableViewCell:tableView];
}

Person模型类创建cell赋值数据
// 返回tableViewCell
- (UITableViewCell *)createdTableViewCell:(UITableView *)tableView;
{
    NSString *cellIdentifier = self.cellClassName;

    BaseCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
    if (!cell) {
         cell = [[[NSClassFromString(cellIdentifier) class] alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentifier];
    }
    [cell setPerson:self];
    return cell;
}

**结论：**
模型创建cell，控制器基本没有代码处理，更加简洁

