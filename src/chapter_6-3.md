# アプリケーションを実装

以下を実装

1. サーバーレスポンスを受け取るための型定義
2. 記事のリスト表示の実装
3. アプリケーションの起点の実装

- `use_state`
    
    コンポーネント内での状態管理に使う
    
    呼び出し時に初期値を与えると、その戻り値として`UseStateHandle<T>`型の値が返される
    
    `set`メソッドを持ち、コンポーネント内の任意の場所で値を変更可能
    
- `use_effect_with_deps`
    
    コンポーネントに対して副作用を与える際に用いる副作用フック
    
    依存している値に変更があった場合、中が実行される
    
    今回はAPIを叩いて取得した値をpostsに入れる処理を行なっている
    
    簡単な説明は以下
    
    [RustのWasmで仮想DOMなYewの紹介とyew-style-in-rsの使い方](https://zenn.dev/matcha_choco010/articles/2022-06-30-yew-introduction#use_effect_with_deps)
    

- 実際のコード
    
    ```rust
    use gloo_net::http::Request;
    use serde::Deserialize;
    use yew::prelude::*;
    
    // APIからのレスポンスを受け取るために、3章で作ったPost構造体を使う
    // driveアトリビュートを変更する
    #[derive(Deserialize, Clone, PartialEq)]
    pub struct Post {
        id: i32,
        title: String,
        body: String,
        published: bool,
    }
    
    // Post構造体のベクタと、クリック時のコールバックを受け取る構造体を定義
    #[derive(Properties, PartialEq)]
    struct PostsListProps {
        posts: Vec<Post>,
        on_click: Callback<Post>,
    }
    
    // APIのレスポンスを取得してidとタイトル一覧を表示するコンポーネント
    #[function_component(PostsList)]
    fn posts_list(
        PostsListProps { posts, on_click }: &PostsListProps,
    ) -> Html {
        posts
            .iter()
            .map(|post| {
                let on_post_select = {
                    let on_click = on_click.clone();
                    let post = post.clone();
                    Callback::from(move |_| {
                        on_click.emit(post.clone())
                    })
                };
    
                html! {
                    <p onclick={on_post_select}>{
                        format!("{}: {}", post.id, post.title)
                    }</p>
                }
            })
            .collect()
    }
    
    #[derive(Clone, Properties, PartialEq)]
    struct PostsDetailProps {
        post: Post,
    }
    
    // 記事詳細を表示するコンポーネント
    #[function_component(PostDetail)]
    fn post_detail(
        PostsDetailProps { post }: &PostsDetailProps,
    ) -> Html {
        html! {
            <div>
                <h3>{ post.title.clone() }</h3>
                <p>{ post.body.clone() }</p>
            </div>
        }
    }
    
    // postのリストと選択されたpostを状態として持てるよう
    #[function_component(App)]
    fn app() -> Html {
        // postのリストの状態を持つ
        let posts = use_state(|| vec![]);
        {
            let posts = posts.clone();
            // フックを使用してAPIからデータを取得してpostsの状態を更新する
            use_effect_with_deps(
                move |_| {
                    wasm_bindgen_futures::spawn_local(
                        async move {
                            let fetched_posts: Vec<Post> =
                                Request::get("/posts")
                                    .send()
                                    .await
                                    .unwrap()
                                    .json()
                                    .await
                                    .unwrap();
                            posts.set(fetched_posts);
                        },
                    );
                    || ()
                },
                (),
            );
        }
    
        // 選択されたpostの状態を持つ
        let selected_post = use_state(|| None);  
        let on_post_select = {
            let selected_post = selected_post.clone();
            // postリストの項目がクリックされた時に内容を表示できるよう
            // on_clickのコールバックを内でselected_postの状態を更新する
            Callback::from(move |post: Post| {
                selected_post.set(Some(post))
            })
        };
        let detail = selected_post.as_ref().map(|post| {
            html! {
                <PostDetail post={post.clone()} />
            }
        });
    
        html! {
            <>
                <h1>{ "My blog" }</h1>
                <div>
                    <h3>{ "posts list" }</h3>
                    <PostsList posts={(*posts).clone()}
                        on_click={on_post_select.clone()} />
                </div>
                { for detail }
            </>
        }
    }
    
    fn main() {
        yew::Renderer::<App>::new().render();
    }
    ```

### バックエンドと合わせて実行 

3章で作成したバックエンドを実行する

`actix-web-blog`フォルダにて

```bash
$ cargo make --env-fike .env run
```

フロントエンドを別ターミナルで起動

今回のバックエンドはcorsの設定をしていないので、trunkのプロキシ機能を通じてアクセス

```bash
$ trunk serve --port 8081 \
--proxy-backend=http://localhost:8080/posts
```

結果

![localhost_8081.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/642182cc-da97-45ae-a186-d02908e26ce5/localhost_8081.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221214%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221214T050557Z&X-Amz-Expires=86400&X-Amz-Signature=9e33e20bbbd31d8b72899f8fd1fb11a2c7c8b9fa2dbcebe5c5941e6725621383&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22localhost_8081.png%22&x-id=GetObject)

![localhost_8081.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9227b361-1b35-4899-bd04-df883f31ff7f/localhost_8081.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221214%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221214T050607Z&X-Amz-Expires=86400&X-Amz-Signature=7a08637f2662edc897b68c7ac97e424ea0ae3d4343c78ac0ae6b5383fe5ef7eb&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22localhost_8081.png%22&x-id=GetObject)