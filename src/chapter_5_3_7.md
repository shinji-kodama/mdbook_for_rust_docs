# 記事取得のAPI実装
## GETリクエストで記事を取得するAPIを実装

repository.rsに以下を追加
- 記事一覧取得（`list_posts`）
- 記事詳細取得（`get_post`）
```rust
// repository.rs

impl Repository {
    pub fn new(database_url: &str) // (省略)
    pub async fn create_post(&self, new_post: NewPost) // (省略)

		
    pub async fn list_posts(
        &self,
    ) -> Result<Vec<Post>, ApiError> {
        let mut conn = self.pool.get()?;
        let res = web::block(move || {
            posts::table.load(&mut conn)
        })
        .await??;

        Ok(res)
    }

    pub async fn get_post(
        &self,
        id: i32,
    ) -> Result<Post, ApiError> {
        let mut conn = self.pool.get()?;
        let res = web::block(move || {
            posts::table
                .find(id)
                .first(&mut conn)
                .optional()
        })
        .await??
        .ok_or(ApiError::NotFound)?;

        Ok(res)
    }
}
```