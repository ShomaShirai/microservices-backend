# バックエンドのMVC設計(GO)

## Model
cmd/modelsの部分で表示されている，型定義を行う部分

## View
基本的にHTMLテンプレートがこの部分に該当されているが，APIでは，WebAPIが返すJSONレスポンスを表す
```go
type Account struct {
    Id      int `json:"id"`
    FoodId  int `json:"food_id"`
    DrinkId int `json:"drink_id"`
}
```

## Controller
cmd/controllerの部分で表示されている，HTTPリクエストを受け取る部分

## 依存性の方向
Web APIの設計では，依存性の逆転は無しで，ユーザーから離れている順に依存性の方向を決めているので，テストを行いにくい
```bash
HTTPリクエスト
  ↓
Router
  ↓
Controller
  ↓
Model
  ↓
Database
```
