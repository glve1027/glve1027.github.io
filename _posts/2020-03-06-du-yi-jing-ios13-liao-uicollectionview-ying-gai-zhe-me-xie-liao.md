---
layout:       post
title:        "都已经iOS13了，UICollectionView应该这么写了！"
author:       "GH"
header-style: text
catalog:      true
tags:
    - Swift 
    - Object-C
---

### 最近看到[ios ray](https://www.kodeco.com/home)一篇关于iOS新特性的一篇文章，感觉还是挺有收获的，以此记录一下，iOS13 新出的一些关于UICollectionView最新的API使用的问题：

#### 1. 概念
> iOS13, 重新梳理的关于CollectionView的一些概念性的东西，我觉得这个是非常核心的以此重构设计。现在整个UICollectionView分为下面三级结构: Section > Group > Item (层次以此减小，解释：一个Section包含一个或者多个Group, 一个Group包含多个Item)

#### 2. 关于如何设置比例，尺寸
> 其实这个问题，还是挺重要的，它可以节省很多不必要的逻辑和代码判断，这边引入了一个NSCollectionLayoutDimension, 通过类的实例化，可以实现三种尺寸：

* absolute 【绝对尺寸】
* estimated 【这是一个预估的尺寸，会更加内容实际的尺寸，会变化】
* fractionalWidth 【尺寸和宽度的比例，有效值0~1，例如0.2，就是这个尺寸，是父层级宽度的0.2倍，！！注意这里不是屏幕尺寸，而是父层级】

> 结合上面两个，配置UICollectionViewLayout用的是一个最新的子类UICollectionViewCompositionalLayout

```swift
// Swift版本
func configureLayout() -> UICollectionViewCompositionalLayout {
    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.2), heightDimension: .fractionalHeight(1.0))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = NSDirectionalEdgeInsets(top: 0, leading: 1, bottom: 2, trailing: 1)
    
    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .fractionalWidth(0.2))
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])
    
    let section = NSCollectionLayoutSection(group: group)
    return UICollectionViewCompositionalLayout(section: section)
}

// OC版本
- (UICollectionViewCompositionalLayout *)configureLayout {
    NSCollectionLayoutSize *itemSize = [NSCollectionLayoutSize sizeWithWidthDimension:[NSCollectionLayoutDimension fractionalWidthDimension:0.2] heightDimension:[NSCollectionLayoutDimension fractionalHeightDimension:1.0]];
    NSCollectionLayoutItem *item = [NSCollectionLayoutItem itemWithLayoutSize:itemSize];
    item.contentInsets = NSDirectionalEdgeInsetsMake(5, 5, 5, 5);

    NSCollectionLayoutSize *groupSize = [NSCollectionLayoutSize sizeWithWidthDimension:[NSCollectionLayoutDimension fractionalWidthDimension:1.0] heightDimension:[NSCollectionLayoutDimension fractionalWidthDimension:0.2]];
    NSCollectionLayoutGroup *group = [NSCollectionLayoutGroup horizontalGroupWithLayoutSize:groupSize subitems:@[item]];
    
    NSCollectionLayoutSection *section = [NSCollectionLayoutSection sectionWithGroup:group];
    
    return [[UICollectionViewCompositionalLayout alloc] initWithSection:section];
}
```

#### 3. 使用

> 关于使用，iOS这次摒弃了使用代理的方式来进行配置，而是创建一个新对象，持有UICollectionView，返回Block给用户来配置的方式，这样确实简便很多，看代码：

```swift
// Swift版本
func configureDatasource() {
        collectionDataSource = UICollectionViewDiffableDataSource<Section, Int>(collectionView: self.collectionView) {
            (collect, indexPath, number) -> UICollectionViewCell? in
            
            guard let cell = collect.dequeueReusableCell(withReuseIdentifier: NumberCell.reuseIdentifier, for: indexPath) as? NumberCell else {
                fatalError("Cannot create new cell")
            }
            cell.label.text = number.description
            return cell
        }
        
        var initialSnapshot = NSDiffableDataSourceSnapshot<Section, Int>()
        initialSnapshot.appendSections([.main])
        initialSnapshot.appendItems(Array(1...100), toSection: .main)
        
        collectionDataSource.apply(initialSnapshot, animatingDifferences: false)
    }
// OC版本
- (void)configureDatasource {
    NSDiffableDataSourceSnapshot *initialSnapshot = [NSDiffableDataSourceSnapshot new];
    [initialSnapshot appendSectionsWithIdentifiers:@[@"main"]];
    
    NSMutableArray *datas = [NSMutableArray new];
    for (int i = 0 ; i < 100; i++) {
        [datas addObject:[NSNumber numberWithInt:i]];
    }
    [initialSnapshot appendItemsWithIdentifiers:[datas copy]];
    
    [self.dataSource applySnapshot:initialSnapshot animatingDifferences:NO completion:nil];
}

- (UICollectionViewDiffableDataSource<NSString *,NSNumber *> *)dataSource {
    if (_dataSource == nil) {
        _dataSource = [[UICollectionViewDiffableDataSource alloc] initWithCollectionView:self.collectionView cellProvider:^UICollectionViewCell * _Nullable(UICollectionView * _Nonnull collect, NSIndexPath * _Nonnull index, id _Nonnull obj) {
            
            NumberCell *cell = [collect dequeueReusableCellWithReuseIdentifier:NSStringFromClass([NumberCell class]) forIndexPath:index];
            cell.label.text = [obj description];
            
            return cell;
        }];
    }
    return _dataSource;
}
```

#### 4. 关于类NSDiffableDataSourceSnapshot的一些理解：

* 就相当于装饰器模式一样，对同一份数据源，我们可以经过不同的加工操作，得到我们想要的数据。
* 这样做的另一个操作，就是我们不需要，把这些耗时的操作，放到需要的数据的时候才去处理。
* 第三部分就是关于名字叫Diff，我猜想对于同一份数据，在内存尽量小的前提下，能够高效的处理数据【！！猜测】