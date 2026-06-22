# DayNavi ER図 設計

## 1. ER図の概要

DayNaviでは、ユーザーごとに予定、カテゴリ、リマインド設定、AI提案履歴を管理する。

Version 1では、以下のテーブルを中心に実装する。

* `auth_user`
* `schedules_category`
* `schedules_schedule`
* `schedules_user_reminder_setting`

Version 3以降で、AIによる予定・タスク作成補助のために以下を追加する。

* `schedules_ai_suggestion`

---

## 2. テーブル一覧

| テーブル名                             | 概要                 | 実装Version |
| --------------------------------- | ------------------ | --------- |
| `auth_user`                       | Django標準のユーザー情報    | Version 1 |
| `schedules_category`              | ユーザーごとの予定カテゴリ      | Version 1 |
| `schedules_schedule`              | 予定情報               | Version 1 |
| `schedules_user_reminder_setting` | ユーザーごとのデイリーリマインド設定 | Version 1 |
| `schedules_ai_suggestion`         | AIによる予定・タスク提案履歴    | Version 3 |

---

## 3. draw.io用 テーブル定義

### 3.1 auth_user

```text
auth_user
-------------------------
PK id
UK username
email
password
is_active
is_staff
is_superuser
last_login
date_joined
```

### 説明

| カラム名         | 制約 | 説明            |
| ------------ | -- | ------------- |
| id           | PK | ユーザーID        |
| username     | UK | ユーザー名         |
| email        |    | メールアドレス       |
| password     |    | ハッシュ化されたパスワード |
| is_active    |    | 有効ユーザーか       |
| is_staff     |    | 管理画面にログインできるか |
| is_superuser |    | 管理者権限を持つか     |
| last_login   |    | 最終ログイン日時      |
| date_joined  |    | 登録日時          |

---

### 3.2 schedules_category

```text
schedules_category
-------------------------
PK id
FK user_id
name
color
created_at
updated_at
```

### 説明

| カラム名       | 制約 | 説明              |
| ---------- | -- | --------------- |
| id         | PK | カテゴリID          |
| user_id    | FK | カテゴリを作成したユーザーID |
| name       |    | カテゴリ名           |
| color      |    | 表示色             |
| created_at |    | 作成日時            |
| updated_at |    | 更新日時            |

### 補足

同じユーザー内で同じカテゴリ名を重複登録できないようにする。

```text
UK user_id + name
```

例：

```text
坂本さんの「就活」カテゴリ → OK
坂本さんがもう一度「就活」を作成 → NG
別ユーザーの「就活」カテゴリ → OK
```

---

### 3.3 schedules_schedule

```text
schedules_schedule
-------------------------
PK id
FK user_id
FK category_id
title
date
start_time
end_time
reminder_time
memo
is_done
created_at
updated_at
```

### 説明

| カラム名          | 制約 | 説明            |
| ------------- | -- | ------------- |
| id            | PK | 予定ID          |
| user_id       | FK | 予定を登録したユーザーID |
| category_id   | FK | カテゴリID        |
| title         |    | 予定タイトル        |
| date          |    | 予定日           |
| start_time    |    | 開始時刻          |
| end_time      |    | 終了時刻          |
| reminder_time |    | 予定ごとの確認時刻     |
| memo          |    | メモ            |
| is_done       |    | 完了状態          |
| created_at    |    | 作成日時          |
| updated_at    |    | 更新日時          |

### 補足

`category_id` はNULL許可にする。

理由は、カテゴリを削除しても予定自体は削除しないためである。
カテゴリが削除された予定は、カテゴリ未設定として扱う。

---

### 3.4 schedules_user_reminder_setting

```text
schedules_user_reminder_setting
-------------------------
PK id
FK UK user_id
daily_reminder_enabled
daily_reminder_time
reminder_method
created_at
updated_at
```

### 説明

| カラム名                   | 制約     | 説明               |
| ---------------------- | ------ | ---------------- |
| id                     | PK     | リマインド設定ID        |
| user_id                | FK, UK | ユーザーID           |
| daily_reminder_enabled |        | デイリーリマインドを有効にするか |
| daily_reminder_time    |        | 今日の予定をまとめて確認する時刻 |
| reminder_method        |        | 通知方法             |
| created_at             |        | 作成日時             |
| updated_at             |        | 更新日時             |

### 補足

`user_id` に `UK` を付ける。

理由は、1人のユーザーにつきリマインド設定は1つだけにするためである。

---

### 3.5 schedules_ai_suggestion

```text
schedules_ai_suggestion
-------------------------
PK id
FK user_id
input_text
suggested_data
is_applied
created_at
```

### 説明

| カラム名           | 制約 | 説明             |
| -------------- | -- | -------------- |
| id             | PK | AI提案ID         |
| user_id        | FK | ユーザーID         |
| input_text     |    | ユーザーが入力した自然文   |
| suggested_data |    | AIが生成した予定・タスク案 |
| is_applied     |    | 提案内容を登録に反映したか  |
| created_at     |    | 作成日時           |

### 補足

AIが生成した内容は直接予定として保存しない。
ユーザーが確認・修正し、確定した場合のみ `schedules_schedule` に登録する。

---

## 4. リレーション一覧

| 親テーブル                | 子テーブル                             | 関係  | 説明                    |
| -------------------- | --------------------------------- | --- | --------------------- |
| `auth_user`          | `schedules_category`              | 1:N | 1人のユーザーは複数のカテゴリを持つ    |
| `auth_user`          | `schedules_schedule`              | 1:N | 1人のユーザーは複数の予定を持つ      |
| `schedules_category` | `schedules_schedule`              | 1:N | 1つのカテゴリは複数の予定に使われる    |
| `auth_user`          | `schedules_user_reminder_setting` | 1:1 | 1人のユーザーは1つのリマインド設定を持つ |
| `auth_user`          | `schedules_ai_suggestion`         | 1:N | 1人のユーザーは複数のAI提案履歴を持つ  |

---

## 5. draw.ioで引く線

draw.ioでは、以下の5本の線を引く。

```text
auth_user.id 1 ───< N schedules_category.user_id
```

```text
auth_user.id 1 ───< N schedules_schedule.user_id
```

```text
schedules_category.id 1 ───< N schedules_schedule.category_id
```

```text
auth_user.id 1 ─── 1 schedules_user_reminder_setting.user_id
```

```text
auth_user.id 1 ───< N schedules_ai_suggestion.user_id
```

---

## 6. draw.ioでの表記例

draw.io上では、次のように簡略表記してもよい。

```text
auth_user 1 ───< schedules_category
```

```text
auth_user 1 ───< schedules_schedule
```

```text
schedules_category 1 ───< schedules_schedule
```

```text
auth_user 1 ─── 1 schedules_user_reminder_setting
```

```text
auth_user 1 ───< schedules_ai_suggestion
```

`<` が付いている側が「多」を表す。

---

## 7. ER図の全体構造

```text
auth_user
├── 1:N schedules_category
│       └── 1:N schedules_schedule
├── 1:N schedules_schedule
├── 1:1 schedules_user_reminder_setting
└── 1:N schedules_ai_suggestion
```

より正確には、`schedules_schedule` は以下の2つに紐づく。

```text
auth_user 1:N schedules_schedule
schedules_category 1:N schedules_schedule
```

---

## 8. Mermaid記法版

GitHubのMarkdown上で簡易的にER図を表示したい場合は、以下も使える。

```mermaid
erDiagram
    auth_user ||--o{ schedules_category : creates
    auth_user ||--o{ schedules_schedule : owns
    schedules_category ||--o{ schedules_schedule : classifies
    auth_user ||--|| schedules_user_reminder_setting : has
    auth_user ||--o{ schedules_ai_suggestion : has

    auth_user {
        int id PK
        string username UK
        string email
        string password
        boolean is_active
        boolean is_staff
        boolean is_superuser
        datetime last_login
        datetime date_joined
    }

    schedules_category {
        int id PK
        int user_id FK
        string name
        string color
        datetime created_at
        datetime updated_at
    }

    schedules_schedule {
        int id PK
        int user_id FK
        int category_id FK
        string title
        date date
        time start_time
        time end_time
        time reminder_time
        text memo
        boolean is_done
        datetime created_at
        datetime updated_at
    }

    schedules_user_reminder_setting {
        int id PK
        int user_id FK_UK
        boolean daily_reminder_enabled
        time daily_reminder_time
        string reminder_method
        datetime created_at
        datetime updated_at
    }

    schedules_ai_suggestion {
        int id PK
        int user_id FK
        text input_text
        json suggested_data
        boolean is_applied
        datetime created_at
    }
```

---

## 9. 制約一覧

| テーブル名                             | カラム名             | 制約 | 説明                          |
| --------------------------------- | ---------------- | -- | --------------------------- |
| `auth_user`                       | `id`             | PK | ユーザーID                      |
| `auth_user`                       | `username`       | UK | ユーザー名は一意                    |
| `schedules_category`              | `id`             | PK | カテゴリID                      |
| `schedules_category`              | `user_id`        | FK | `auth_user.id` を参照          |
| `schedules_category`              | `user_id + name` | UK | 同一ユーザー内でカテゴリ名を一意にする         |
| `schedules_schedule`              | `id`             | PK | 予定ID                        |
| `schedules_schedule`              | `user_id`        | FK | `auth_user.id` を参照          |
| `schedules_schedule`              | `category_id`    | FK | `schedules_category.id` を参照 |
| `schedules_user_reminder_setting` | `id`             | PK | リマインド設定ID                   |
| `schedules_user_reminder_setting` | `user_id`        | FK | `auth_user.id` を参照          |
| `schedules_user_reminder_setting` | `user_id`        | UK | ユーザーごとに1設定のみ                |
| `schedules_ai_suggestion`         | `id`             | PK | AI提案ID                      |
| `schedules_ai_suggestion`         | `user_id`        | FK | `auth_user.id` を参照          |

---

## 10. on_delete の方針

| 関係                         | Djangoでの指定                  | 理由                  |
| -------------------------- | --------------------------- | ------------------- |
| User → Category            | `on_delete=models.CASCADE`  | ユーザー削除時にカテゴリも削除する   |
| User → Schedule            | `on_delete=models.CASCADE`  | ユーザー削除時に予定も削除する     |
| Category → Schedule        | `on_delete=models.SET_NULL` | カテゴリ削除時に予定は残す       |
| User → UserReminderSetting | `on_delete=models.CASCADE`  | ユーザー削除時に設定も削除する     |
| User → AiSuggestion        | `on_delete=models.CASCADE`  | ユーザー削除時にAI提案履歴も削除する |

---

## 11. Djangoモデル案

```python
from django.db import models
from django.contrib.auth.models import User


class Category(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    name = models.CharField(max_length=50)
    color = models.CharField(max_length=20, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        constraints = [
            models.UniqueConstraint(
                fields=["user", "name"],
                name="unique_category_name_per_user",
            )
        ]

    def __str__(self):
        return self.name


class Schedule(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
    )
    title = models.CharField(max_length=100)
    date = models.DateField()
    start_time = models.TimeField(null=True, blank=True)
    end_time = models.TimeField(null=True, blank=True)
    reminder_time = models.TimeField(null=True, blank=True)
    memo = models.TextField(blank=True)
    is_done = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title


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

    def __str__(self):
        return f"{self.user.username} reminder setting"


class AiSuggestion(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    input_text = models.TextField()
    suggested_data = models.JSONField()
    is_applied = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"AI suggestion for {self.user.username}"
```

---

## 12. draw.ioに保存するファイル

draw.ioで作成したER図は以下に保存する。

```text
diagrams/er-diagram.drawio
```

GitHubで表示しやすいように、PNGでも書き出す。

```text
docs/images/er-diagram.png
```

---

## 13. GitHubに追加するコマンド

```powershell
git add diagrams\er-diagram.drawio docs\images\er-diagram.png
git commit -m "Add ER diagram"
git push origin main
```

まだ画像を書き出していない場合は、`drawio` ファイルだけ先に追加してもよい。

```powershell
git add diagrams\er-diagram.drawio
git commit -m "Add ER diagram source"
git push origin main
```
