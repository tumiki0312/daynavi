# DayNavi DB設計書

## 1. 概要

DayNaviでは、ユーザーごとに予定データを管理する。

Version 1では、ユーザー情報、予定情報、デイリーリマインド設定を管理する。  
Version 3以降では、AIによる予定・タスク作成補助機能のためにAI提案履歴を管理する。

---

## 2. テーブル一覧

| テーブル名 | 概要 |
|---|---|
| auth_user | Django標準のユーザー情報 |
| schedules_schedule | 予定情報 |
| schedules_user_reminder_setting | ユーザーごとのデイリーリマインド設定 |
| schedules_ai_suggestion | AIによる予定・タスク提案履歴 |

---

## 3. auth_user

Django標準のUserモデルを使用する。

| カラム名 | 型 | NULL | 説明 |
|---|---|---|---|
| id | integer | NO | ユーザーID |
| username | varchar | NO | ユーザー名 |
| email | varchar | YES | メールアドレス |
| password | varchar | NO | ハッシュ化されたパスワード |
| is_active | boolean | NO | 有効ユーザーかどうか |
| is_staff | boolean | NO | 管理画面にログインできるか |
| date_joined | datetime | NO | 登録日時 |

---

## 4. schedules_schedule

予定情報を管理するテーブル。

| カラム名 | 型 | NULL | 説明 |
|---|---|---|---|
| id | integer | NO | 予定ID |
| user_id | integer | NO | 登録ユーザーID |
| title | varchar(100) | NO | 予定タイトル |
| category | varchar(30) | NO | 予定カテゴリ |
| date | date | NO | 予定日 |
| start_time | time | YES | 開始時刻 |
| end_time | time | YES | 終了時刻 |
| reminder_time | time | YES | 予定ごとの確認時刻 |
| memo | text | YES | メモ |
| is_done | boolean | NO | 完了状態 |
| created_at | datetime | NO | 作成日時 |
| updated_at | datetime | NO | 更新日時 |

### category の値

| 値 | 表示名 |
|---|---|
| job | 就活 |
| study | 学習 |
| development | 開発 |
| school | 大学 |
| part_time | アルバイト |
| private | 個人予定 |
| other | その他 |

---

## 5. schedules_user_reminder_setting

ユーザーごとのデイリーリマインド設定を管理するテーブル。

| カラム名 | 型 | NULL | 説明 |
|---|---|---|---|
| id | integer | NO | 設定ID |
| user_id | integer | NO | ユーザーID |
| daily_reminder_enabled | boolean | NO | デイリーリマインドを有効にするか |
| daily_reminder_time | time | NO | 今日の予定をまとめて確認する時刻 |
| reminder_method | varchar(20) | NO | 通知方法 |
| created_at | datetime | NO | 作成日時 |
| updated_at | datetime | NO | 更新日時 |

### reminder_method の値

| 値 | 表示名 |
|---|---|
| app | アプリ内 |
| email | メール |
| line | LINE |
| discord | Discord |

---

## 6. schedules_ai_suggestion

AIが生成した予定・タスク案を管理するテーブル。  
Version 3で実装予定。

| カラム名 | 型 | NULL | 説明 |
|---|---|---|---|
| id | integer | NO | AI提案ID |
| user_id | integer | NO | ユーザーID |
| input_text | text | NO | ユーザーが入力した自然文 |
| suggested_data | json | NO | AIが生成した予定・タスク案 |
| is_applied | boolean | NO | 提案内容を登録に反映したか |
| created_at | datetime | NO | 作成日時 |

---

## 7. リレーション

| 親テーブル | 子テーブル | 関係 |
|---|---|---|
| auth_user | schedules_schedule | 1対多 |
| auth_user | schedules_user_reminder_setting | 1対1 |
| auth_user | schedules_ai_suggestion | 1対多 |

---

## 8. Djangoモデル案

```python
from django.db import models
from django.contrib.auth.models import User


class Schedule(models.Model):
    CATEGORY_CHOICES = [
        ("job", "就活"),
        ("study", "学習"),
        ("development", "開発"),
        ("school", "大学"),
        ("part_time", "アルバイト"),
        ("private", "個人予定"),
        ("other", "その他"),
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=100)
    category = models.CharField(max_length=30, choices=CATEGORY_CHOICES, default="other")
    date = models.DateField()
    start_time = models.TimeField(null=True, blank=True)
    end_time = models.TimeField(null=True, blank=True)
    reminder_time = models.TimeField(null=True, blank=True)
    memo = models.TextField(blank=True)
    is_done = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)


class UserReminderSetting(models.Model):
    REMINDER_METHOD_CHOICES = [
        ("app", "アプリ内"),
        ("email", "メール"),
        ("line", "LINE"),
        ("discord", "Discord"),
    ]

    user = models.OneToOneField(User, on_delete=models.CASCADE)
    daily_reminder_enabled = models.BooleanField(default=True)
    daily_reminder_time = models.TimeField(default="07:00")
    reminder_method = models.CharField(
        max_length=20,
        choices=REMINDER_METHOD_CHOICES,
        default="app",
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)


class AiSuggestion(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    input_text = models.TextField()
    suggested_data = models.JSONField()
    is_applied = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)