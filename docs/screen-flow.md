# DayNavi 画面遷移図

## 1. 概要

DayNaviの画面遷移を整理する。  
画面遷移図はFigmaで作成し、画像として本ドキュメントに掲載する。

## 2. Figmaリンク

Figmaで作成した画面遷移図のリンクを以下に記載する。

- Figma URL: <!-- ここにFigmaの共有リンクを貼る -->

## 3. 画面遷移図

![画面遷移図](./images/screen-flow.png)

## 4. Version 1 画面一覧

| 画面名 | URL案 | 概要 |
|---|---|---|
| トップページ | / | アプリ概要、ログイン・登録への導線 |
| ユーザー登録画面 | /accounts/signup/ | 新規ユーザー登録 |
| ログイン画面 | /accounts/login/ | ユーザーログイン |
| 今日の予定画面 | /schedules/today/ | 今日の予定一覧 |
| 予定一覧画面 | /schedules/ | 自分の予定一覧 |
| 日付別予定画面 | /schedules/date/<date>/ | 指定日の予定一覧 |
| 予定詳細画面 | /schedules/<id>/ | 予定詳細 |
| 予定登録画面 | /schedules/new/ | 予定登録 |
| 予定編集画面 | /schedules/<id>/edit/ | 予定編集 |
| 予定削除確認画面 | /schedules/<id>/delete/ | 予定削除確認 |
| リマインド設定画面 | /settings/reminder/ | デイリーリマインド設定 |

## 5. 主な画面遷移

### 5.1 未ログイン時

```text
トップページ
├── ユーザー登録画面
└── ログイン画面