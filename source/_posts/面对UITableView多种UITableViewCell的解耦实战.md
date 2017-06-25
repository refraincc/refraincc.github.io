---
title: 面对UITableView多种UITableViewCell的解耦实战
layout: post
date:  
author: "refraincc"
tags:
	- Coder
---

# 面对UITableView多种UITableViewCell的解耦实战

最近项目中有一个界面需要展示多种Cell,而且在业务需求上也会有多种变化，Cell的种类有可能非常的多，UI差异也比较大,数据结构也不一样，这种结构有可能有5种，也有可能有10种往上。

如果按照以往的方法，为每种Cell写注册，高度，创建的代码，是一种非常吃力不讨好的办法，最终导致Controller中的代码会越来越臃肿，越来越难以维护！

那么我们应该如何解决这种问题呢？如何通过对原来的代码最少的改动的基础上进行UI的调整，添加？

## 前期的准备工作

首先在最初的与后端接口设计的时候，我们选择了一种十分通用的Json格式:
```bash
Array [
			Dictionary{
				uitype : @"1001"
				dict : {
							title : @"",
							name : @""
						}
			}
			Dictionary{
				uitype : @"1002"
				dict : {
							list : [1, 2, 3]
						}
			}
			Dictionary{
				uitype : @"1003"
				dict : {
							content : @""
						}
				
			}
		]
```
以上的`Json格式`中我们可以看出在，数组里的字典只有两个字段，一个是`uitpye`,一个是`dict`,其中`uitype`是为了决定Cell的类型，和之后为了将`dict`转换成与`uitype`想对应的`Model`源数据。

这样的`Json格式`可以方便我们在将不同的源数据转换成TableView想要的Cell和Cell展示数据需要的Model

好了前期的准备工作好了，那么接下来就让我们开始下面的工作吧~

## 基本Model数据

面对`TableView`的`dataSource`和`delegate`最重要的两个代理方法
```bash
	- (CGFloat) tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    	return cellHeight;
	}

	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
	{
	    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:kCellIdentifier];
	    return cell;
	}
```
我们要考虑如何用最简洁的代码告诉`TableView` ，`Cell的高度`、`Cell的类型`；


为了最简单直接的实现这两个代码方法，我们需要为`TableView`创建一个`ModelManager`；
	```bash
	typedef NS_ENUM(NSInteger, ModelType) {
	    ModelTypeApple      = 1001,
	    ModelTypeBanana     = 1002,
	    ModelTypeOrange     = 1003
	};
	
	@protocol ModelDelegate <NSObject>
	
	/**
	 通过dataModel拿到CellHeight
	 */
	- (CGFloat) cellModelHeight;
	
	
	@end
	
	@protocol CellDelegate <NSObject>
	
	/**
	 cell通过此方法进行刷新数据
	 */
	- (void) refreshDataModel:(id<ModelDelegate>)dataModel;
	
	@end
	
	
	@interface ModelManager : NSObject
	
	/*后端返回的源数据*/
	@property (nonatomic, assign)ModelType uitype;
	
	@property (nonatomic, copy)NSDictionary *dict;
	
	
	/*辅助属性*/
	@property (nonatomic, copy)NSString *dataModelClassName; //uitype对应的dataModel
	
	@property (nonatomic, copy)NSString *cellClassName; //uitype对应的Cell
	
	@property (nonatomic, assign)CGFloat cellHeight; //dataModel通过ModelManagerDelegate返回的cell高度
	
	@property (nonatomic,strong)id<ModelDelegate> dataModel; //通过uitype将dict转换的model实例
	
	
	@end
```

这样我们就创建了一个通用的Model,也是拿到后端数据后创建的一个最原始的Model,通过这个Model我们可以创建出`TableView`需要的Cell类型、CellHeight、Model。

通过这个基础的ModelManager，我们可以很简洁的实现`TableView`的代理方法.
```bash
	
	- (NSInteger) tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
	    return self.dataSource.count;
	}
	
	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
	
	    ModelManager *modelMgr = self.dataSource[indexPath.row];
	    
	    id<CellDelegate> cell = [tableView dequeueReusableCellWithIdentifier:modelMgr.cellClassName forIndexPath:indexPath];
	    
	    if (!cell) {
	        
	        Class cellClass = NSClassFromString(modelMgr.cellClassName);
	        
	        [tableView registerClass:cellClass forCellReuseIdentifier:modelMgr.cellClassName];
	    }
	    
	    if ([cell respondsToSelector:@selector(refreshDataModel:)]) {
	        [cell refreshDataModel:modelMgr.dataModel];
	    }
	    return (UITableViewCell *)cell;
	}
	
	- (CGFloat) tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
	    ModelManager *modelMgr = self.dataSource[indexPath.row];
	    
	    return modelMgr.cellHeight;
	}
```
可以看到在控制器的`TableView`代理方法中，我们并不需要写太多的代码，并且在之后的项目维护中，我们也不用再去动这部分代码。

## 创建dataModel
为了给调取模型数据的计算做缓存处理，所以以下的创建都是通过`lazy`的方式
	```bash	
	@implementation ModelManager
	
	
	- (NSString *)cellClassName{
	    
	    if (!_cellClassName) {
	        
	        NSString *cellClassName = @"";
	        
	        switch (self.uitype) {
	            case ModelTypeApple:
	                cellClassName = @"AppleTableViewCell";
	                break;
	                
	            case ModelTypeBanana:
	                cellClassName = @"BananaTableViewCell";
	                break;
	                
	            case ModelTypeOrange:
	                cellClassName = @"OrangeTableViewCell";
	                break;
	                
	            default:
	                break;
	        }
	        
	        _cellClassName = cellClassName;
	    }
	    
	    return _cellClassName;
	}
	
	
	- (NSString *)dataModelClassName{
	    if (!_dataModelClassName) {
	        
	        NSString *dataModelClassName = @"";
	        
	        switch (self.uitype) {
	            case ModelTypeApple:
	                dataModelClassName = @"AppleTableViewCell";
	                break;
	                
	            case ModelTypeBanana:
	                dataModelClassName = @"BananaTableViewCell";
	                break;
	                
	            case ModelTypeOrange:
	                dataModelClassName = @"OrangeTableViewCell";
	                break;
	                
	            default:
	                break;
	        }
	        
	        _dataModelClassName = dataModelClassName;
	    }
	    
	    return _dataModelClassName;
	}
	
	- (id<ModelDelegate>)dataModel{
	    if (!_dataModel) {
	        
	        Class dataModelClass = NSClassFromString(self.dataModelClassName);
	        id<ModelDelegate> dataModel = [[dataModelClass alloc]init];
	        //id<ModelDelegate> dataModel = [dataModelClass objectWithKeyValues:self.dict];
	        _dataModel = dataModel;
	    }
	    return _dataModel;
	}
	
	
	- (CGFloat)cellHeight{
	    if (!_cellHeight) {
	        if ([self.dataModel respondsToSelector:@selector(cellModelHeight)]) {
	            _cellHeight = [self.dataModel cellModelHeight];
	        }
	    }
	    return _cellHeight;
	}
	
	@end

```
这样在`tableView`执行`heightForRowAtIndexPath`方法的时候，`dataModel`就可以在`ModelManager`中通过`uitype`创建出来。而我们只用在`id<ModelManager>`的`dataModel`中返回`CellHeight`。

这下`Model`层基本已经的核心代码就这样了。接下来的就是`TableViewCell`的核心代码了

## TableViewCell的数据获取
```bash

	#import "ModelManager.h"
	
	@interface AppleTableViewCell : UITableViewCell <CellDelegate>
	
	- (void)refreshDataModel:(id<ModelDelegate>)dataModel;
	
	@end

```
在创建任意的`UITableViewCell`中，我们只要遵守`CellDelegate`并实现`	- (void)refreshDataModel:(id<ModelDelegate>)dataModel;`方法就可以获取数据源。

## 总结
在这套设计方案中，`Controller`中的代码基本是不用再改动的。我们只需要在`ModelManager`中通过`dataModelClassName`和`cellClassName`来添加源数据类型就可以了，可以解决由于多种`TableViewCell`类型展示的问题。


