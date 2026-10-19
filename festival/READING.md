# 読解ノート：自分のアプリのコード3箇所

> **氏名：** 中島 多笑
> **AI利用レベル：** 1（質問のみ。説明は自分の言葉で書く）
> **最終更新：** 2026-10-05

生成AIに書いてもらったコードでも、自分で書いたコードでもかまいません。自分のアプリの中から3箇所を選んで、次の3つの問いに答えてください。分からないところは、生成AIに「この行は何をしていますか」「この行を消すとどうなりますか」と聞いてかまいません。ただし、書くのは自分の言葉です。

> 💡 担任の先生の指示で、項目が増えることがあります。

---

## 箇所1：QRコードを生成・表示する処理

```swift
if let qrImage = QRCodeGenerator.generate(
    from: url.absoluteString
) {
    Image(uiImage: qrImage)
        .interpolation(.none)
        .resizable()
        .scaledToFit()
        .frame(width: 220, height: 220)
        .padding(20)
        .background(Color.white)
} else {
    Text("QRコードを作成できませんでした。")
}
```

**何をしているか：**
Web名刺のURLをQRコードに変換し、画面に表示する処理です。
QRCodeGenerator.generate() という関数にURLを渡してQRコード画像を生成しています。
生成に成功した場合は画像を表示し、失敗した場合はエラーメッセージを表示します。

**なぜこう書くのか：**
if let を使っている理由は、QRコードの生成に失敗する可能性があるためです。
QRCodeGenerator.generate() は、生成に失敗した場合に nil を返す設計だと考えられます。
そのため、if let を使用して、画像が生成された場合だけ表示するようにしています。
また、.interpolation(.none) は、QRコードを拡大したときに画像がぼやけるのを防ぐためです。

**もしこう書かなかったら：**
if let を使用せず、生成結果を強制的に取り出すと、QRコードの生成に失敗した場合にアプリがクラッシュする可能性があります。
また、.interpolation(.none) を指定しない場合は、QRコードの輪郭がぼやけて読み取りにくくなる可能性があります。

---

## 箇所2：写真をSupabaseにアップロードする処理

```swift
if let photoData = participant.photoData,
   Date() < publicEnd,
   participant.webPhotoToken == nil {

    let path: String

    if let savedPath = participant.webPhotoPath {
        path = savedPath
    } else {
        path = try await PhotoUploadService.uploadPhoto(
            photoData: photoData
        )

        participant.webPhotoPath = path
        try modelContext.save()
    }
}
```

**何をしているか：**
撮影した写真が存在し、公開期限内で、写真共有用のトークンがまだない場合に、写真のアップロード処理を進めています。
すでにアップロード済みの写真がある場合は、保存先のパスを再利用します。

**なぜこう書くのか：**
if let を使用することで、写真が存在する場合だけ処理できます。
また、Date() < publicEnd によって公開期限を確認しています。
try await は、エラーが発生する可能性のある非同期処理を実行するために使用しています。

**もしこう書かなかったら：**
写真が存在しない場合でもアップロード処理を実行しようとしたり、同じ写真を何度もアップロードしたりする可能性があります。
また、await を適切に使用しないと、非同期関数を正しく呼び出せません。
---

## 箇所3：エラーが発生した場合の処理

```swift
do {
    guard let url = CardURLBuilder.makeURL(
        from: participant
    ) else {
        errorMessage = "名刺URLを作成できませんでした。"
        return
    }

    cardURL = url

} catch {
    errorMessage =
        "名刺の準備に失敗しました。\n\(error.localizedDescription)"
}
```

**何をしているか：**
Web名刺のURLを生成し、成功した場合は cardURL に保存しています。
URLを生成できなかった場合は、エラーメッセージを設定して処理を終了します。
また、写真アップロードなどでエラーが発生した場合は、catch でエラーを受け取ります。

**なぜこう書くのか：**
guard let は、必要な値が存在することを確認し、存在しない場合に処理を終了するために使用します。
do / catch は、try を使用した処理で発生したエラーに対応するためです。

**もしこう書かなかったら：**
URLが生成できなかった場合でも後続の処理を進めようとしてしまう可能性があります。
また、適切なエラー処理がないと、失敗した原因を利用者に伝えられなくなります。
---

## 自分ではまだ説明できない箇所（正直に）

（アプリの中で「動いているけれど何をしているか分からない」ところを書く。ここに書いたものは、後期の文法の授業で分かるようになります。）
