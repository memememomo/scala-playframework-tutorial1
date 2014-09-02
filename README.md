Scala+Play チュートリアル

フレームを作成する。

```bash:shell
$ activator new products
$ rm products/public/images/favicon.png
```

activatorを起動する。

```bash:shell
$ cd products
$ activator
[products] $ 
```

IntelliJ用のファイルを作成する。

```bash::shell
[products] $ idea
```

Twitter BootstrapをDLする。
http://getbootstrap.com/2.3.2/

以下のファイルを *public/stylesheets/* に置く。

- docs/assets/css/bootstrap.css

以下のファイルを *public/images* に置く。

- img/glyphicons-halflings-white.png
- img/glyphicons-halflings.png

以下の内容の *public/stylesheets/main.css* を置く。

```css:public/stylesheets/main.css
body { color:black; }
        body, p, label { font-size:15px; }
        .label { font-size:13px; line-height:16px; }
        .alert-info { border-color:transparent; background-color:#3A87AD;
            color:white; font-weight:bold; }
        div.screenshot { width: 800px; margin:20px; background-color:#D0E7EF; }
        .navbar-fixed-top .navbar-inner { padding-left:20px; }
        .navbar .nav > li > a { color:#bbb; }
.screenshot > .container { width: 760px; padding: 20px; }
.navbar-fixed-top, .navbar-fixed-bottom { position:relative; }
h1 { font-size:125%; }
table { border-collapse: collapse; width:100%; }
th, td { text-align:left; padding: 0.3em 0;
    border-bottom: 1px solid white; }
tr.odd td { }
form { float:left; margin-right: 1em; }
legend { border: none; }
fieldset > div { margin: 12px 0; }
.help-block { display: inline; vertical-align: middle; }
.error .help-block { display: none; }
.error .help-inline { padding-left: 9px; color: #B94A48; }
footer { clear: both; text-align: right; }
dl.products { margin-top: 0; }
dt { clear: right; }
.barcode { float:right; margin-bottom: 10px; border: 4px solid white; }
```

言語ファイルを作成する。

```conf/messages
application.name = Product catalog
```

Modelを作成する。

```app/models/Product.scala
package models


case class Product(ean: Long, name: String, description: String)

object Product {
  var products = Set(
    Product(5010255079763L, "Paperclips Large",
      "Large Plain Pack of 1000"),
    Product(5018206244666L, "Giant Paperclips",
      "Giant Plain 51mm 100 pack"),
    Product(5018306332812L, "Paperclip Giant Plain",
      "Giant Plain Pack of 10000"),
    Product(5018306312913L, "No Tear Paper Clip",
      "No Tear Extra Large Pack of 1000"),
    Product(5018206244611L, "Zebra Paperclips",
      "Zebra Length 28mm Assorted 150 Pack")
  )

  def findAll = products.toList.sortBy(_.ean)
}
```

HTMLテンプレートを作成する。


```app/views/products/list.scala.html
@(products: List[Product])(implicit lang: Lang)
@main(Messages("application.name")) {
  <dl class="products">
    @for(product <- products) {
       <dt>@product.name</dt>
      <dd>@product.description</dd>
} </dl>
}
```

```app/views/main.scala.html
@(title: String)(content: Html)(implicit lang: Lang)

<!DOCTYPE html>
<html>
    <head>
        <title>@title</title>
        <link rel="stylesheet" media="screen" href='@routes.Assets.at("stylesheets/bootstrap.css")'>
        <link rel="stylesheet" media="screen" href="@routes.Assets.at("stylesheets/main.css")">
    </head>
    <body class="screenshot">
        <div class="navbar navbar-fixed-top">
            <div class="navbar-inner">
                <div class="container">
                    <a class="brand" href="@routes.Application.index()">
                        @Messages("application.name")
                    </a>
                </div>
            </div>
        </div>

        <div class="container">
        @content
        </div>
    </body>
</html>
```

Controllerを作成する。

```app/controllers/Products.scala
package controllers

import play.api.mvc.{Action, Controller}
import models.Product

object Products extends Controller {
  def list = Action { implicit request =>
    val products = Product.findAll
    Ok(views.html.products.list(products))
  }
}
```

ルーティング設定を行う。

```conf/routes
# Home page
GET        /                    controllers.Application.index
GET        /products            controllers.Products.list

# Map static resources from the /public folder to the /assets URL path
GET        /assets/*file        controllers.Assets.at(path="/public", file)
```

トップページのリダイレクト処理を行う。

```scala:app/controllers/Application.scala
package controllers

import play.api._
import play.api.mvc._

object Application extends Controller {

  def index = Action {
    Redirect(routes.Products.list())
  }
  
}
```

デバッグ用テンプレートを作成する。

```html:app/views/debug.scala.html
@()(implicit lang: Lang)
@import play.api.Play.current
<footer>
    lang = @lang.code,
    user = @current.configuration.getString("environment.user"),
    date = @(new java.util.Date().format("yyyy-MM-dd HH:mm"))
</footer>
```

設定を追加する。

```conf/application.conf
environment.user=${USER}
```

テンプレートにデバッグ読み込み処理を追加する。

```html:app/views/main.scala.html
<div class="container">
  @content
  @debug()
</div>
```

## バーコード表示機能を追加

Modelにメソッドを追加する。

```app/models/Product.scala
def findByEan(ean: Long) = products.find(_.ean == ean)
```

詳細ページ用テンプレートを作成する。

```html:app/views/products/details.scala.html
@(product: Product)(implicit lang: Lang)

@main(Messages("products.details", product.name)) {
    <h2>
        @tags.barcode(product.ean)
        @Messages("products.details", product.name)
    </h2>

    <dl class="dl-horizontal">
        <dt>@Messages("ean"):</dt>
        <dd>@product.ean</dd>

        <dt>@Messages("name"):</dt>
        <dd>@product.name</dd>

        <dt>@Messages("description"):</dt>
        <dd>@product.description</dd>
    </dl>
}
```

バーコード用テンプレートを作成する。

```html:app/views/tags/barcode.scala.html
@(ean: Long)
<img class="barcode" alt="@ean" src="@routes.Barcodes.barcode(ean)">
```

言語ファイルにデータを追加する。

```conf/messages
ean = EAN
name = Name
description = Description

products.details = Product: {0}
```

Controllerにメソッドを追加する。

```scala:app/controllers/Products.scala
  def show(ean: Long) = Action { implicit request =>
    Product.findByEan(ean).map { product =>
      Ok(views.html.products.details(product))
    }.getOrElse(NotFound)
  }
```

ルーティング設定を追加する。

```scala:conf/routes
GET        /products/:ean        controllers.Products.show(ean: Long)
```

依存モジュールを追加する。

```scala:build.sbt
  "net.sf.barcode4j" % "barcode4j" % "2.0"
```

リロードして、プロジェクトファイルを作り直す。

```bash:shell
[products] $ reload
[products] $ idea
```

Controllerを追加する。

```scala:app/controllers/Barcodes.scala
package controllers

import play.api.mvc.{Action, Controller}

object Barcodes extends Controller {

  val ImageResolution = 144

  def barcode(ean: Long) = Action {

    import java.lang.IllegalArgumentException

    val MimeType = "image/png"
    try {
      val imageData = ean13BarCode(ean, MimeType)
      Ok(imageData).as(MimeType)
    }
    catch {
      case e: IllegalArgumentException =>
        BadRequest("Couldn't generate bar code. Error: " + e.getMessage)
    }
  }

  def ean13BarCode(ean: Long, mimeType: String): Array[Byte] = {

    import java.io.ByteArrayOutputStream
    import java.awt.image.BufferedImage
    import org.krysalis.barcode4j.output.bitmap.BitmapCanvasProvider
    import org.krysalis.barcode4j.impl.upcean.EAN13Bean

    val output: ByteArrayOutputStream = new ByteArrayOutputStream
    val canvas: BitmapCanvasProvider = new BitmapCanvasProvider(output, mimeType, ImageResolution, BufferedImage.TYPE_BYTE_BINARY, false, 0)

    val barcode = new EAN13Bean()
    barcode.generateBarcode(canvas, String valueOf ean)
    canvas.finish

    output.toByteArray
  }
}
```

バーコードデータを生成するControllerへのルーティング設定を行う。

```conf/routes
GET /barcode/:ean controllers.Barcodes.barcode(ean: Long)
```


## 新規作成フォーム

言語ファイルにデータを追加する。

```conf/messages
products.form = Product details
products.new = (new)
products.new.command = New
products.new.submit = Add
products.new.success = Successfully added product {0}.

validation.errors = Please correct the errors in the form.
validation.ean.duplicate = A product with this EAN code already exists
```

Controllerにimportするモジュールを追加する。

```scala:app/controllers/Products.scala
import play.api.data.Form
import play.api.data.Forms.{mapping, longNumber, nonEmptyText}
import play.api.i18n.Messages
```

