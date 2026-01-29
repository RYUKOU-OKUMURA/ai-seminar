# Google Apps Script - Docs/Drive/Calendar/Forms Service

## 概要

Google Apps Script provides four powerful services for interacting with Google Docs, Google Drive, Google Calendar, and Google Forms. This guide covers the essential methods and practical patterns for each service.

---

## DocsService (DocumentApp)

### 基本的な操作

#### ドキュメントの作成と取得

```javascript
// 新しいドキュメントを作成
var doc = DocumentApp.create('新しいドキュメント');

// IDで既存のドキュメントを開く
var doc = DocumentApp.openById('DOCUMENT_ID');

// アクティブなドキュメントを取得（コンテナバインドスクリプト）
var doc = DocumentApp.getActiveDocument();
```

#### ドキュメントの基本メソッド

| メソッド | 説明 |
|----------|------|
| `getName()` / `setName(name)` | タイトルの取得/設定 |
| `getId()` | ドキュメントIDの取得 |
| `getUrl()` | ドキュメントのURLを取得 |
| `getBlob()` | ドキュメントをBlobとして取得 |
| `getBody()` | ドキュメントのボディセクションを取得 |
| `addEditor(email)` | 編集者を追加 |
| `addViewer(email)` | 閲覧者を追加 |
| `saveAndClose()` | ドキュメントを保存して閉じる |

### コンテンツの操作

#### テキストと段落の追加

```javascript
var body = doc.getBody();

// 段落の追加
body.appendParagraph('見出しテキスト')
    .setHeading(DocumentApp.ParagraphHeading.HEADING1);

// テキストの追加
var text = body.appendParagraph('通常のテキスト');

// テキストのスタイル設定
text.editAsText()
    .setBold(false)
    .setForegroundColor('#ff0000')
    .setFontFamily('Roboto')
    .setFontSize(14);
```

#### リストの作成

```javascript
// 箇条書きリスト
body.appendListItem('項目1');
body.appendListItem('項目2')
    .setGlyphType(DocumentApp.GlyphType.BULLET);

// 番号付きリスト
body.appendListItem('番号項目1')
    .setGlyphType(DocumentApp.GlyphType.NUMBER);
```

#### 表の作成と操作

```javascript
// 2次元配列から表を作成
var table = body.appendTable([
    ['列1', '列2', '列3'],
    ['データ1', 'データ2', 'データ3'],
    ['データ4', 'データ5', 'データ6']
]);

// 表のスタイル設定
table.setBorderColor('#cccccc')
      .setBorderWidth(1);

// セルの操作
var cell = table.getCell(1, 1); // 2行2列
cell.setBackgroundColor('#ffff00')
    .setVerticalAlignment(DocumentApp.VerticalAlignment.CENTER);

// 行の追加
table.appendTableRow();
```

#### 画像の挿入

```javascript
// URLから画像を挿入
var image = body.appendImage(UrlFetchApp.fetch('IMAGE_URL'));

// Blobから画像を挿入
var blob = DriveApp.getFileById('FILE_ID').getBlob();
body.appendImage(blob);

// 画像サイズの設定
image.setWidth(300).setHeight(200);

// 配置画像（固定位置）
var paragraph = body.appendParagraph('');
var positionedImage = paragraph.addPositionedImage(blob);
positionedImage.setTopOffset(10).setLeftOffset(10);
```

#### ページブレイクと区切り線

```javascript
body.appendPageBreak();
body.appendHorizontalRule();
```

### 要素の検索と置換

```javascript
// テキストの検索
var searchResult = body.findText('検索テキスト');

// 正規表現での検索
var pattern = '\\d{4}-\\d{2}-\\d{2}';
var result = body.findText(pattern);

// テキストの置換
body.replaceText('古いテキスト', '新しいテキスト');

// 正規表現での置換
body.replaceText('\\b\\w+@\\w+\\.\\w+\\b', 'メールアドレス');
```

### Attribute一覧

| Attribute | 説明 |
|-----------|------|
| `BACKGROUND_COLOR` | 背景色 |
| `BOLD` | 太字 |
| `FOREGROUND_COLOR` | 文字色 |
| `FONT_FAMILY` | フォントファミリー |
| `FONT_SIZE` | フォントサイズ |
| `ITALIC` | 斜体 |
| `UNDERLINE` | 下線 |
| `STRIKETHROUGH` | 取り消し線 |
| `LINK_URL` | リンクURL |

### ParagraphHeading一覧

| 定数 | 説明 |
|------|------|
| `NORMAL` | 通常テキスト |
| `HEADING1` | 見出し1 |
| `HEADING2` | 見出し2 |
| `HEADING3` | 見出し3 |
| `TITLE` | タイトル |
| `SUBTITLE` | サブタイトル |

---

## DriveService (DriveApp)

### 基本的な操作

#### ファイルの作成と取得

```javascript
// テキストファイルの作成
var file = DriveApp.createFile('ファイル名.txt', 'ファイルの内容');

// MIMEタイプを指定して作成
var file = DriveApp.createFile('report.csv', csvContent, 'text/csv');

// Blobから作成
var blob = Utilities.newBlob(data, mimeType, fileName);
var file = DriveApp.createFile(blob);

// ファイルの取得
var file = DriveApp.getFileById('FILE_ID');

// 名前でファイルを検索
var files = DriveApp.getFilesByName('ファイル名');
while (files.hasNext()) {
    var file = files.next();
    // 処理
}
```

#### フォルダの操作

```javascript
// フォルダの作成
var folder = DriveApp.createFolder('新しいフォルダ');

// ルートフォルダの取得
var rootFolder = DriveApp.getRootFolder();

// フォルダ内にファイルを作成
folder.createFile('file.txt', 'content');

// サブフォルダの作成
var subFolder = folder.createFolder('サブフォルダ');

// フォルダの移動
file.moveTo(folder);
```

### ファイル・フォルダの検索

```javascript
// すべてのファイルを取得
var files = DriveApp.getFiles();
while (files.hasNext()) {
    var file = files.next();
    console.log(file.getName());
}

// MIMEタイプで検索
var spreadsheets = DriveApp.getFilesByType(MimeType.GOOGLE_SHEETS);

// 検索クエリを使用
var files = DriveApp.searchFiles('title contains "報告書" and trashed=false');

// フォルダ内のファイルを検索
var filesInFolder = folder.searchFiles('mimeType contains "image/"');

// ゴミ箱内のファイル
var trashedFiles = DriveApp.getTrashedFiles();
```

### 共有と権限

```javascript
// 編集者の追加
file.addEditor('user@example.com');
folder.addEditors(['editor1@example.com', 'editor2@example.com']);

// 閲覧者の追加
file.addViewer('viewer@example.com');

// 権限の取り消し
file.removeEditor('user@example.com');

// 共有設定
file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
file.setSharing(DriveApp.Access.PRIVATE, DriveApp.Permission.NONE);

// 所有者の変更
file.setOwner('newowner@example.com');

// 編集者による共有を許可/禁止
file.setShareableByEditors(false);
```

### Access列挙型

| 定数 | 説明 |
|------|------|
| `ANYONE` | 誰でもアクセス可能 |
| `ANYONE_WITH_LINK` | リンクを知っている誰でもアクセス可能 |
| `DOMAIN` | 同一ドメインのユーザーがアクセス可能 |
| `DOMAIN_WITH_LINK` | 同一ドメインでリンクを知っているユーザーがアクセス可能 |
| `PRIVATE` | 明示的に権限を付与されたユーザーのみアクセス可能 |

### Permission列挙型

| 定数 | 説明 |
|------|------|
| `VIEW` | 閲覧のみ |
| `EDIT` | 編集可能 |
| `COMMENT` | コメントのみ可能 |
| `OWNER` | 所有者 |
| `ORGANIZER` | 整理権限（共有ドライブ） |
| `FILE_ORGANIZER` | ファイル整理権限（共有ドライブ） |
| `NONE` | 権限なし |

### ファイルメタデータ

```javascript
// 基本情報
var name = file.getName();
var id = file.getId();
var url = file.getUrl();
var size = file.getSize(); // バイト単位
var mimeType = file.getMimeType();

// 日付情報
var createdDate = file.getDateCreated();
var updatedDate = file.getLastUpdated();

// 説明の設定
file.setDescription('このファイルの説明');
var description = file.getDescription();

// スター付与
file.setStarred(true);
var isStarred = file.isStarred();

// ゴミ箱操作
file.setTrashed(true);
var isTrashed = file.isTrashed();
```

### フォルダメタデータ

```javascript
// フォルダ情報
var name = folder.getName();
var id = folder.getId();
var url = folder.getUrl();
var size = folder.getSize();

// 親フォルダの取得
var parents = folder.getParents();

// フォルダ内のアイテム
var files = folder.getFiles();
var subFolders = folder.getFolders();
```

---

## CalendarService (CalendarApp)

### 基本的な操作

#### カレンダーの取得

```javascript
// デフォルトカレンダーを取得
var calendar = CalendarApp.getDefaultCalendar();

// すべてのカレンダーを取得
var calendars = CalendarApp.getAllCalendars();

// 名前でカレンダーを検索
var calendars = CalendarApp.getCalendarsByName('仕事');

// IDでカレンダーを取得
var calendar = CalendarApp.getCalendarById('calendar_id@group.calendar.google.com');

// 新しいカレンダーを作成
var newCalendar = CalendarApp.createCalendar('プロジェクト管理');
```

### イベントの作成

#### 通常のイベント

```javascript
var calendar = CalendarApp.getDefaultCalendar();

// 時間指定イベント
var startTime = new Date('2026-01-30T10:00:00');
var endTime = new Date('2026-01-30T11:00:00');
var title = '会議';

var event = calendar.createEvent(title, startTime, endTime);

// オプション付きイベント作成
var event = calendar.createEvent('チームミーティング',
    new Date('2026-01-30T14:00:00'),
    new Date('2026-01-30T15:00:00'), {
        description: '今週の進捗確認',
        location: '会議室A',
        guests: 'taro@example.com,hanako@example.com',
        sendInvites: true
    });
```

#### 終日イベント

```javascript
// 単日の終日イベント
var date = new Date('2026-01-30');
var event = calendar.createAllDayEvent('休日', date);

// 複数日の終日イベント
var startDate = new Date('2026-02-01');
var endDate = new Date('2026-02-05');
var event = calendar.createAllDayEvent('長期休暇', startDate, endDate);
```

#### 説明からのイベント作成

```javascript
// 自然言語形式からイベントを作成
var event = CalendarApp.createEventFromDescription(
    '会議 2026年1月30日 14:00から15:00'
);
```

### 繰り返しイベント

```javascript
// 繰り返しルールの作成
var recurrence = CalendarApp.newRecurrence()
    .addWeeklyRule()
    .onlyOnWeekdays(CalendarApp.Weekday.MONDAY, CalendarApp.Weekday.WEDNESDAY)
    .until(new Date('2026-06-30'));

// 繰り返しイベントの作成
var eventSeries = calendar.createEventSeries('定例会議',
    new Date('2026-02-01T10:00:00'),
    new Date('2026-02-01T11:00:00'),
    recurrence,
    { location: '会議室B' }
);
```

### イベントの取得と検索

```javascript
// 期間内のイベントを取得
var startTime = new Date('2026-01-01T00:00:00');
var endTime = new Date('2026-01-31T23:59:59');
var events = calendar.getEvents(startTime, endTime);

// 特定のイベントを取得
var event = calendar.getEventById('EVENT_ID');

// 特定の日のイベントを取得
var date = new Date('2026-01-30');
var events = calendar.getEventsForDay(date);

// オプション付きでイベントを取得
var events = calendar.getEvents(startTime, endTime, {
    search: '会議' // タイトルまたは説明に含まれるテキスト
});
```

### イベントの操作

```javascript
// イベント情報の取得
var title = event.getTitle();
var description = event.getDescription();
var location = event.getLocation();
var startTime = event.getStartTime();
var endTime = event.getEndTime();

// イベントの更新
event.setTitle('新しいタイトル');
event.setDescription('更新された説明');
event.setLocation('新しい場所');

// ゲストの操作
var guests = event.getGuestList();
for (var i = 0; i < guests.length; i++) {
    var guest = guests[i];
    console.log(guest.getEmail() + ': ' + guest.getGuestStatus());
}

// ゲストの追加
event.addGuest('newguest@example.com');

// イベントの削除
// 単一のイベントを削除
event.deleteEvent();

// 繰り返しイベントの削除
eventSeries.deleteEventSeries();
```

### カレンダーの設定

```javascript
// カレンダー情報
var name = calendar.getName();
var timeZone = calendar.getTimeZone();
var isPrimary = calendar.isMyPrimaryCalendar();
var isHidden = calendar.isHidden();

// カレンダーの更新
calendar.setName('新しい名前');
calendar.setTimeZone('Asia/Tokyo');
calendar.setDescription('カレンダーの説明');
calendar.setColor('#ff0000');

// 表示/非表示の設定
calendar.setSelected(true); // UIに表示
calendar.setHidden(false);
```

### Color列挙型

| 定数 | 説明 |
|------|------|
| `BLUE` | 青 (#2952A3) |
| `RED` | 赤 (#A32929) |
| `GREEN` | 緑 (#0D7813) |
| `YELLOW` | 黄 (#AB8B00) |
| `ORANGE` | オレンジ (#BE6D00) |
| `PURPLE` | 紫 (#7A367A) |

### EventType列挙型

| 定数 | 説明 |
|------|------|
| `DEFAULT` | 通常のイベント |
| `BIRTHDAY` | 誕生日 |
| `FOCUS_TIME` | フォーカスタイム |
| `OUT_OF_OFFICE` | 休暇 |

---

## FormsService (FormApp)

### 基本的な操作

#### フォームの作成と取得

```javascript
// 新しいフォームを作成
var form = FormApp.create('アンケートフォーム');
var form = FormApp.create('非公開フォーム', false); // 非公開で作成

// 既存のフォームを開く
var form = FormApp.openById('FORM_ID');
var form = FormApp.openByUrl('FORM_URL');

// アクティブなフォームを取得（コンテナバインドスクリプト）
var form = FormApp.getActiveForm();
```

### アイテム（質問）の追加

#### テキスト質問

```javascript
// 短文テキスト
var textItem = form.addTextItem();
textItem.setTitle('お名前を入力してください')
       .setHelpText('フルネームで入力してください')
       .setRequired(true);

// 段落テキスト
var paragraphItem = form.addParagraphTextItem();
paragraphItem.setTitle('ご意見・ご要望')
            .setHelpText('自由に記述してください')
            .setRequired(false);
```

#### 選択肢質問

```javascript
// チェックボックス（複数選択）
var checkboxItem = form.addCheckboxItem();
checkboxItem.setTitle('興味のあるトピックを選択してください（複数）')
    .setChoices([
        checkboxItem.createChoice('テクノロジー'),
        checkboxItem.createChoice('ビジネス'),
        checkboxItem.createChoice('デザイン'),
        checkboxItem.createChoice('マーケティング')
    ])
    .setRequired(true);

// ラジオボタン（単一選択）
var multipleChoiceItem = form.addMultipleChoiceItem();
multipleChoiceItem.setTitle('性別')
    .setChoiceValues(['男性', '女性', 'その他'])
    .showOtherOption(true) // 「その他」オプションを表示
    .setRequired(true);

// リスト（ドロップダウン）
var listItem = form.addListItem();
listItem.setTitle('居住地')
    .setChoiceValues(['北海道', '東北', '関東', '中部', '近畿', '中国', '四国', '九州'])
    .setRequired(true);
```

#### グリッド質問

```javascript
// チェックボックスグリッド
var checkboxGrid = form.addCheckboxGridItem();
checkboxGrid.setTitle('各カテゴリの評価（複数選択可）')
    .setRows(['品質', '価格', 'サービス'])
    .setColumns(['満足', '普通', '不満']);

// ラジオボタングリッド
var gridItem = form.addGridItem();
gridItem.setTitle('製品評価（1つ選択）')
    .setRows(['デザイン', '機能', '使いやすさ'])
    .setColumns(['良い', '普通', '悪い']);
```

#### 日付・時間質問

```javascript
// 日付
var dateItem = form.addDateItem();
dateItem.setTitle('予約日')
      .setRequired(true);

// 日時
var dateTimeItem = form.addDateTimeItem();
dateTimeItem.setTitle('イベント日時')
           .setRequired(true);

// 時間
var timeItem = form.addTimeItem();
timeItem.setTitle('開始時間')
       .setRequired(true);

// 所要時間
var durationItem = form.addDurationItem();
durationItem.setTitle('所要時間');
```

#### その他のアイテム

```javascript
// 評価（レーティング）
var ratingItem = form.addRatingItem();
ratingItem.setTitle('満足度')
         .setRatingIconType(FormApp.RatingIconType.STAR)
         .setRatingScaleBounds(1, 5);

// スケール
var scaleItem = form.addScaleItem();
scaleItem.setTitle('年齢層')
       .setBounds(1, 10)
       .setLabels('10代', '60代以上');

// セクションヘッダー
var sectionHeader = form.addSectionHeaderItem();
sectionHeader.setTitle('個人情報');

// 画像アイテム
var imageItem = form.addImageItem();
imageItem.setImage(UrlFetchApp.fetch('IMAGE_URL'))
         .setAlignment(FormApp.Alignment.CENTER);

// ビデオアイテム
var videoItem = form.addVideoItem();
videoItem.setVideoUrl('YOUTUBE_VIDEO_URL')
         .setAlignment(FormApp.Alignment.CENTER);

// ページブレイク
var pageBreak = form.addPageBreakItem();
pageBreak.setTitle('次のページへ')
        .setGoToPage(FormApp.PageNavigationType.CONTINUE);
```

### ItemType列挙型

| 定数 | 説明 |
|------|------|
| `TEXT` | 短文テキスト |
| `PARAGRAPH_TEXT` | 段落テキスト |
| `MULTIPLE_CHOICE` | ラジオボタン |
| `CHECKBOX` | チェックボックス |
| `LIST` | ドロップダウンリスト |
| `GRID` | ラジオボタングリッド |
| `CHECKBOX_GRID` | チェックボックスグリッド |
| `DATE` | 日付 |
| `DATETIME` | 日時 |
| `TIME` | 時間 |
| `DURATION` | 所要時間 |
| `RATING` | 評価 |
| `SCALE` | スケール |
| `IMAGE` | 画像 |
| `VIDEO` | 動画 |
| `SECTION_HEADER` | セクションヘッダー |
| `PAGE_BREAK` | ページブレイク |

### バリデーション

```javascript
// テキストバリデーション
var textValidation = FormApp.createTextValidation()
    .setHelpText('メールアドレスを入力してください')
    .requireTextMatchesPattern('^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$')
    .build();

var textItem = form.addTextItem();
textItem.setTitle('メールアドレス')
       .setValidation(textValidation);

// 段落テキストバリデーション
var paragraphValidation = FormApp.createParagraphTextValidation()
    .setHelpText('100文字以上で入力してください')
    .requireTextLengthGreaterThanOrEqualTo(100)
    .build();

var paragraphItem = form.addParagraphTextItem();
paragraphItem.setTitle('ご意見')
            .setValidation(paragraphValidation);

// 数値バリデーション
var numberValidation = FormApp.createParagraphTextValidation()
    .setHelpText('1-100の数字を入力してください')
    .requireNumberBetween(1, 100)
    .build();
```

### 回答の取得

```javascript
// すべての回答を取得
var form = FormApp.openById('FORM_ID');
var responses = form.getResponses();

// 回答の統計情報
var responseCount = form.getResponses().length;

// 各回答の処理
for (var i = 0; i < responses.length; i++) {
    var response = responses[i];
    var respondentEmail = response.getRespondentEmail();
    var timestamp = response.getTimestamp();
    var itemResponses = response.getItemResponses();

    for (var j = 0; j < itemResponses.length; j++) {
        var itemResponse = itemResponses[j];
        var item = itemResponse.getItem();
        var title = item.getTitle();
        var answer = itemResponse.getResponse();
        console.log(title + ': ' + answer);
    }
}

// 最新の回答のみ取得
var latestResponses = form.getResponses(0, 10);
```

### フォームの設定と公開

```javascript
// フォーム情報
var title = form.getTitle();
var description = form.getDescription();
var isPublished = form.isPublished();
var isAcceptingResponses = form.isAcceptingResponses();

// 設定の更新
form.setTitle('アンケート2026')
    .setDescription('このアンケートへのご協力をお願いします')
    .setConfirmationMessage('ご回答ありがとうございます！')
    .setAllowResponseEdits(true)
    .setPublishingSummary(true);

// 回答の宛先を設定
var spreadsheet = SpreadsheetApp.create('回答保存用');
form.setDestination(FormApp.DestinationType.SPREADSHEET, spreadsheet.getId());

// 公開設定
form.setPublished(true);
form.addPublishedEditor('editor@example.com');
form.addPublishedViewer('viewer@example.com');

// URLの取得
var formUrl = form.getPublishedUrl();
var editUrl = form.getEditUrl();
```

### クイズ機能

```javascript
// クイズとして設定
form.setIsQuiz(true);

// 正解とフィードバックの設定
var multipleChoiceItem = form.addMultipleChoiceItem();
multipleChoiceItem.setTitle('JavaScriptの作成者は？')
    .setChoiceValues(['Brendan Eich', 'James Gosling', 'Guido van Rossum'])
    .setRequired(true);

// 正解の設定
var choices = multipleChoiceItem.getChoices();
choices[0].setIsCorrect(true); // Brendan Eich

// フィードバックの設定
var correctFeedback = FormApp.createQuizFeedback()
    .setDisplayText('正解！')
    .build();
multipleChoiceItem.setFeedbackForCorrect(correctFeedback);

var incorrectFeedback = FormApp.createQuizFeedback()
    .setDisplayText('不正解です。Brendan Eichが作成しました。')
    .build();
multipleChoiceItem.setFeedbackForIncorrect(incorrectFeedback);

// 配点設定
multipleChoiceItem.setPoints(10);
```

---

## サービス間連携パターン

### パターン1: フォーム回答をドキュメントに生成

```javascript
function generateDocumentFromForm() {
    var form = FormApp.openById('FORM_ID');
    var responses = form.getResponses();
    var latestResponse = responses[responses.length - 1];

    // ドキュメント作成
    var doc = DocumentApp.create('回答報告書');
    var body = doc.getBody();

    // タイトル
    body.appendParagraph('回答報告書')
        .setHeading(DocumentApp.ParagraphHeading.HEADING1);

    // 回答内容をドキュメントに追加
    var itemResponses = latestResponse.getItemResponses();
    for (var i = 0; i < itemResponses.length; i++) {
        var itemResponse = itemResponses[i];
        body.appendParagraph(itemResponse.getItem().getTitle() + ':')
            .setHeading(DocumentApp.ParagraphHeading.HEADING3);
        body.appendParagraph(itemResponse.getResponse());
    }

    // Driveに移動
    var folder = DriveApp.getFolderById('FOLDER_ID');
    var file = DriveApp.getFileById(doc.getId());
    file.moveTo(folder);

    doc.saveAndClose();
}
```

### パターン2: カレンダーイベントからドキュメント作成

```javascript
function createMinutesFromEvent() {
    var calendar = CalendarApp.getDefaultCalendar();
    var now = new Date();
    var events = calendar.getEvents(
        new Date(now.getFullYear(), now.getMonth(), now.getDate()),
        new Date(now.getFullYear(), now.getMonth(), now.getDate() + 1)
    );

    for (var i = 0; i < events.length; i++) {
        var event = events[i];
        var doc = DocumentApp.create(event.getTitle() + ' 議事録');
        var body = doc.getBody();

        // ヘッダー情報
        body.appendParagraph(event.getTitle())
            .setHeading(DocumentApp.ParagraphHeading.HEADING1);
        body.appendParagraph('日時: ' + event.getStartTime());
        body.appendParagraph('場所: ' + event.getLocation());
        body.appendHorizontalRule();

        body.appendParagraph('出席者:');
        var guests = event.getGuestList();
        for (var j = 0; j < guests.length; j++) {
            body.appendListItem(guests[j].getEmail());
        }

        doc.saveAndClose();
    }
}
```

### パターン3: スプレッドシートデータをフォームで収集

```javascript
function createFormFromSpreadsheet() {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('データ');
    var data = sheet.getDataRange().getValues();

    var form = FormApp.create('データ収集フォーム');

    // ヘッダー行をスキップ
    for (var i = 1; i < data.length; i++) {
        var row = data[i];
        var questionTitle = row[0]; // A列: 質問タイトル
        var questionType = row[1];  // B列: 質問タイプ

        switch (questionType) {
            case 'テキスト':
                form.addTextItem().setTitle(questionTitle);
                break;
            case '段落':
                form.addParagraphTextItem().setTitle(questionTitle);
                break;
            case '選択肢':
                var choices = row[2].split(','); // C列: 選択肢（カンマ区切り）
                form.addMultipleChoiceItem()
                    .setTitle(questionTitle)
                    .setChoiceValues(choices);
                break;
        }
    }

    Logger.log('フォームURL: ' + form.getPublishedUrl());
}
```

### パターン4: Driveファイル整理スクリプト

```javascript
function organizeDriveFiles() {
    var folder = DriveApp.getFolderById('SOURCE_FOLDER_ID');
    var targetFolder = DriveApp.getFolderById('TARGET_FOLDER_ID');
    var files = folder.getFiles();

    var doc = DocumentApp.create('ファイル整理レポート');
    var body = doc.getBody();

    body.appendParagraph('ファイル整理レポート')
        .setHeading(DocumentApp.ParagraphHeading.HEADING1);

    while (files.hasNext()) {
        var file = files.next();
        var name = file.getName();
        var mimeType = file.getMimeType();

        // レポートに記録
        body.appendParagraph('ファイル名: ' + name);
        body.appendParagraph('タイプ: ' + mimeType);

        // 条件に応じて移動
        if (mimeType === MimeType.GOOGLE_DOCS) {
            file.moveTo(targetFolder);
            body.appendParagraph('→ ドキュメント用フォルダに移動');
        } else if (mimeType === MimeType.GOOGLE_SHEETS) {
            file.moveTo(targetFolder);
            body.appendParagraph('→ スプレッドシート用フォルダに移動');
        }

        body.appendHorizontalRule();
    }

    doc.saveAndClose();
}
```

---

## 参考情報

- [Document Service Reference](https://developers.google.com/apps-script/reference/document)
- [Drive Service Reference](https://developers.google.com/apps-script/reference/drive)
- [Calendar Service Reference](https://developers.google.com/apps-script/reference/calendar)
- [Forms Service Reference](https://developers.google.com/apps-script/reference/forms)
