import json
import datetime
import os

### サブスク情報を保存したJSONファイル
DATA_FILE = "subscriptions.json"

def load_subs():
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, "r", encoding="utf-8") as f:
        return json.load(f)

def show_summary():
    subs = load_subs()
    if not subs:
        print("登録されているサブスクリプションはありません。")
        return

    today = datetime.date.today()
    total_monthly = 0
    active_subs = []
    upcoming_alerts = []

    for sub in subs:
        if sub.get("status") == "joined":
            active_subs.append(sub)
            total_monthly += sub.get("monthly_fee", 0)
            
            # 次回更新日の確認
            next_date_str = sub.get("next_payment")
            if next_date_str:
                next_date = datetime.datetime.strptime(next_date_str, "%Y-%m-%d").date()
                days_left = (next_date - today).days
                # 更新日まで7日以内のものをアラート対象にする
                if 0 <= days_left <= 7:
                    upcoming_alerts.append((sub, days_left))

    print("======================================")
    print(" 📊 サブスク棚卸しレポート (Android版)")
    print("======================================")
    print(f"💰 毎月の合計金額 : {total_monthly:,}円")
    print(f"📦 現在加入中の数 : {len(active_subs)}件")
    print("--------------------------------------")
    
    # 加入中一覧 (金額が高い順に並べ替え)
    for sub in sorted(active_subs, key=lambda x: x.get("monthly_fee", 0), reverse=True):
        name = sub.get("name")
        fee = sub.get("monthly_fee", 0)
        next_date = sub.get("next_payment", "未定")
        print(f"・{name:<15} : {fee:>5,}円/月 (次回: {next_date})")

    # 元ツールの「要確認」機能を再現した更新アラート
    if upcoming_alerts:
        print("\n⚠️ 要確認：まもなく更新されるサブスク")
        for sub, days in sorted(upcoming_alerts, key=lambda x: x[1]):
            day_str = "今日!!" if days == 0 else f"あと{days}日"
            print(f"  [!] {sub['name']} ({day_str})")
            print(f"      解約手順: {sub.get('cancel_steps', '不明')}")
            print(f"      解約URL : {sub.get('cancel_url', '不明')}")
    else:
        print("\n✅ 1週間以内に更新されるサブスクはありません。")
    print("======================================")

if __name__ == "__main__":
    show_summary()
