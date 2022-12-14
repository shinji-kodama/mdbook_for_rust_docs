# Cでのメモリ管理
- GCがなくてもメモリの確保と解放を言語側で決定するためメモリ安全
      
      （`C/C++`では `malloc/free`とか`new/delete`でメモリの確保と解放を手動で行なっていた）
        
        
        こんな感じ
        
        ```cpp

        // 文字列を新規メモリ領域に複製する
        char *strdup(const char *src) { 
           size_t len = strlen(src); 
           char *dest = malloc((len + 1) * sizeof( char));   // ここでメモリ確保
           strncpy(dest, src, len + 1); 
        
           return dest; 
        } 
        
        int main(void) { 
           char *buf = strdup("hello"); 
           printf("%s\n", buf);
           free(buf);                                       // ここでメモリ解放
        
           return 0;
        }
        ```