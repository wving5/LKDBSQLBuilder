# LKDBSQLBuilder (derived from LKDBSearchHelper)


#### #Note
* According that Moblie App usually doesn't have complicated SQL queries, following SQL expression are supported: 
  * `select`, `delete`, `where`, `orderBy`, `groupBy`, `limit`, `offset`, `or`,`and`
  * most comparision operator (excluding non-essential ones like `between`, `exists`...)
* v2.0 + Method Chaining
  * try to keep the same expressive as raw SQL 


#### #Install

~~`pod 'LKDBSearchHelper', '~> 1.4'`~~
Instead, pls do it manually...

### #Requirements

* iOS 7.0+
* ARC only 
* FMDB([https://github.com/ccgus/fmdb](https://github.com/ccgus/fmdb))
* LKDBHelper-SQLite-ORM([https://github.com/li6185377/LKDBHelper-SQLite-ORM](https://github.com/li6185377/LKDBHelper-SQLite-ORM))


#### #SQL `where` condition builder


```objective-c
    LKSQLCompositeCondition *cond;

    // single condition
    cond = LKSQLCompositeCondition.clause.where.eq(@"colA", @0.0000001);
    cond = LKSQLCompositeCondition.clause.where.like(@"colA", @"0.1");
    cond = LKSQLCompositeCondition.clause.where.inStrs(@"colA", @[@"str1", @"str2", @"str3"]);
    cond = LKSQLCompositeCondition.clause.where.inNums(@"colA", @[@1, @2, @3]);
    // use `nil` for SQLite `null`
    cond = LKSQLCompositeCondition.clause.where.eq(@"colA", nil);

    // multi condition  ( operator `and` can be omitted 
    cond = LKSQLCompositeCondition.clause
    .where
    .eq(@"colA", nil)
    .or.neq(@"colB", nil)
    .lt(@"colC", nil)
    .and.lte(@"colD", nil)
    .gt(@"colE", nil)
    .or.gte(@"colF", nil)
    .like(@"colG", nil) // 
    .or.isNot(@"colH", nil)
    .inStrs(@"colI", nil)
    .or.inNums(@"colJ", nil);

    // Too Long? Let's use some eye candy
    #define _SQLWhere           LKSQLCompositeCondition.clause
    #define _SQLSelect(_arg_)   LKSQLSelect.clause.from(_arg_.class)
    #define _SQLDelete(_arg_)   LKSQLDelete.clause.from(_arg_.class)

    // Note that `.where` is a candy, it just return `self`. so we omitted it in the macro above to save a method calling

    // now continue...

    // match All
    cond = _SQLWhere.eq(@"colA", nil).matchAll(
        @[
          _SQLWhere.eq(@"colA", nil).neq(@"colAA", nil),
          _SQLWhere.eq(@"colB", nil).neq(@"colBA", nil),
          _SQLWhere.eq(@"colC", nil).neq(@"colCA", nil),
          ]
    );

    // match Any
    cond = _SQLWhere.eq(@"colA", nil).matchAny(
        @[
          _SQLWhere.eq(@"colA", nil).neq(@"colAA", nil),
          _SQLWhere.eq(@"colB", nil).neq(@"colBA", nil),
          _SQLWhere.eq(@"colC", nil).neq(@"colCA", nil),
          ]
    );

    // nested condition
    cond = _SQLWhere.eq(@"colA", nil).and.expr(
        _SQLWhere.eq(@"colAA", nil).or.neq(@"colAB", nil).and.expr(
            _SQLWhere.eq(@"colAAA", nil).or.neq(@"colAAB", nil)
            // allow endless nested conditon if you can ...
        )
    )
    .and.lte(@"colB", nil)
```
     
     
#### #SQL Command

```objective-c
    // SQL: Select
    // exec SQL using `LKDBHelper` according to the class passed in, 
    // which in this case, would be `LKTest` 
    _SQLSelect(LKTest).where(
        _SQLWhere.eq(@"MyAge", @"16")
    ).orderBy(nil).groupBy(nil).limit(0).offset(0).exec();


    // BTW, you can specify `LKDBHelper` if `exec()` not fitting your project scheme
    _SQLSelect(LKTest).where(
      _SQLWhere.eq(@"MyAge", @"16")
      ).orderBy(nil).groupBy(nil).limit(0).offset(0).execIn(_get_using_LKDBHelper_from_somewhere_);

    // Here, orderBy, groupBy should be RAW SQL format
	.orderBy(@"colA DESC, colB ASC") // use `case when X then...` to have more control on sorting if needed
	.groupBy(@"colA, colB")
	// or
	.orderBy(@[@"colA DESC", @"colB ASC"])
	.groupBy(@[@"colA", @"colB"])


    // SQL: Delete
    _SQLDelete(LKTest).where(
        _SQLWhere.eq(@"MyAge", @"16")
    ).exec();

```


#### #SQL Debug Print

`LKSQLSelect` &
`LKSQLDelete` &
`LKSQLCondition` &
`LKSQLCompositeCondition`  all these object could be printed using `- toString`

Also, you can use `FMDatabase+Debug.h` in the DEMO project which intercepting FMDB SQL execution to do same thing ( But NOT full implemented
    

#### #Transaction
    
```objective-c
    LKDBTransaction *transaction = [LKDBSQLite transaction];
    
    [transaction update:obj]; // mark obj as `to be update`
    [transaction insert:obj]; // mark obj as `to be insert` 
    [transaction delete:obj]; // mark obj as `to be delete` 
    
    [transaction batchUpdate:objArr];
    [transaction batchInsert:objArr];
    [transaction batchDelete:objArr];
    
    [transaction execute]; // handle marked object above in transaction
```
    
#### #Co-op with LKDB

LKDB requires raw SQL should be **lowerCase**, you could do that by flip the macro flag `SQL_KEYWORD_UPPERCASE` in `LKDBSQLConstant.h`

Also, you could see some convenience macro there, but **use with caution** though (it may cause problem for Xcode auto-indent / auto-complete, and somehow tricky :)