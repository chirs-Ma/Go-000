学习笔记

问题：
我们在数据库操作的时候，比如 dao 层中当遇到一个 sql.ErrNoRows 的时候，是否应该 Wrap 这个 error，抛给上层。为什么，应该怎么做请写出代码？

分析：
dao层操作数据库可以对应为第三方依赖库，因此这个错误可以wrap这个error抛给上层；但是这个错误属于database/sql特有的，如果直接wrap会造成与上层耦合度增加，为了降低耦合度，我们可以自定义一个错误然后wrap抛给上层


伪代码实现：
```
//dao层

var ErrNotFound = errors.New(" Not found")

func FindArticleByID(ID int) (article *model.Article, err error) {
	 
	row := db.QueryRowContext(ctx, "select id,title,content from articles where id=?", ID)
	err := row.Scan(&article.ID, &article.Title, &article.Content)
	if err == sql.ErrNoRows {
		return nil, errors.Wrap(ErrNotFound, "No corresponding article")
	}
	if err != nil {
		return nil, errors.Wrap(err, "Failed to get article")
	}
	return article, nil
}
```


```
//service层
    articleID := 30
	article, err := dao.FindArticleByID(articleID)
	if errors.Is(err, dao.ErrNotFound) {
		// 返回 404 并打印日志
		log.Printf("404: %v", err)
		return
	}
	if err != nil {
		// 返回 500 并打印日志
		log.Printf("500: %+v", err)
		return
	}
	
```