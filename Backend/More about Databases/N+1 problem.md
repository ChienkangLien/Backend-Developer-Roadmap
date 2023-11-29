# N+1 problem
只要涉及到實體之間的關聯，都可能存在N+1 查詢問題，特別是在使用ORM 框架時。

舉例來說，假設有一個部落格系統，其中有部落客文章（主實體）和對應的作者（關聯實體），如果使用 N+1 查詢：

1. 第一次查詢獲取所有部落客文章的列表（N次查詢）。
2. 接著，對於每篇部落客文章，需要額外進行一次查詢來獲取對應的作者（1次查詢）。
```sql=
-- 獲取所有文章
SELECT * FROM posts;

-- 針對每篇文章獲取對應的作者名字（N次查詢）
SELECT name FROM authors WHERE id = 1;
SELECT name FROM authors WHERE id = 2;
```
*似乎叫做1+N會更貼切*

## ORM
某些 ORM 框架默認使用延遲加載以提高性能，但有時開發者可能在不了解情況下保持了這些默認設置。這可能導致在獲取關聯實體時多次觸發數據庫查詢，增加了額外的負載和查詢次數。

**延遲加載（Lazy Loading）**：即在首次訪問關聯實體時才會觸發額外的數據庫查詢。這意味著，當你獲取主實體列表後，並沒有立即獲取關聯實體的數據，只有在訪問關聯實體時才會執行額外的查詢，從而引發了N+1 查詢問題。
## 解決方式
1. 使用 JOIN：
通過使用 JOIN，一次性獲取主實體和關聯實體的所有數據，避免了額外的單獨查詢。
```sql=
-- 使用 JOIN 獲取所有文章以及對應的作者名字
SELECT p.id, p.title, p.content, a.name AS author_name
FROM posts p
JOIN authors a ON p.author_id = a.id;
```
2. 預加載（Eager Loading）：
在 ORM 框架中，可以使用預加載機制（Eager Loading）一次性獲取所有相關數據。
```java=
// 已Hibernate 為例、可以使用 JOIN FETCH：
List<Post> posts = entityManager.createQuery(
        "SELECT p FROM Post p JOIN FETCH p.author",
        Post.class
).getResultList();
```
3. 批量查詢：
一次性使用 IN 子查詢或者批量查詢來獲取所有相關聯實體的數據。
```sql=
SELECT * FROM authors WHERE id IN (SELECT DISTINCT author_id FROM posts);
```
4. 緩存：
對於某些讀取頻繁但很少變化的數據，可以考慮使用緩存，將已查詢的數據緩存起來，減少額外的數據庫查詢。