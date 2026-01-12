# ouchi-k8s

おうちで Kubernetes を使ってみるためのリポジトリ

## 📦 デプロイ済みアプリケーション

### FreshRSS - RSS/Atom フィードリーダー

ArgoCD を使って FreshRSS をデプロイしています。

#### アーキテクチャ
- **データベース**: SQLite（内蔵、永続化済み）
- **ストレージ**: hostPath（`/var/lib/k8s-volumes/freshrss/`）
- **外部アクセス**: NodePort（30080番ポート）
- **名前空間**: `freshrss`

#### アクセス方法
```
http://<ミニPCのIPアドレス>:30080
```

例: `http://192.168.1.100:30080`

---

## 🚀 デプロイ手順

### 前提条件
- Kubernetes クラスタが稼働していること
- ArgoCD がインストールされていること
- kubectl でクラスタにアクセスできること

### 1. リポジトリをクローン
```bash
git clone https://github.com/a-takamin/ouchi-k8s.git
cd ouchi-k8s
```

### 2. ArgoCD Application を作成
```bash
kubectl apply -f apps/freshrss/application.yaml
```

### 3. 同期状態を確認
```bash
# ArgoCD CLI の場合
argocd app get freshrss

# または kubectl で確認
kubectl get application -n argocd
```

### 4. Pod の起動を確認
```bash
kubectl get pods -n freshrss
```

### 5. アクセス
ブラウザで `http://<ミニPCのIPアドレス>:30080` にアクセス

初回アクセス時に初期設定が表示されます。

---

## 📁 ディレクトリ構成

```
ouchi-k8s/
├── README.md
└── apps/
    └── freshrss/
        ├── application.yaml           # ArgoCD Application 定義
        └── manifests/                 # Kubernetes マニフェスト
            ├── namespace.yaml
            ├── persistentvolume.yaml
            ├── persistentvolumeclaim.yaml
            ├── deployment.yaml
            └── service.yaml
```

---

## 🔧 各ファイルの役割

| ファイル | 役割 |
|---------|------|
| `application.yaml` | ArgoCD にデプロイを指示する設定ファイル |
| `namespace.yaml` | freshrss 専用の名前空間を作成 |
| `persistentvolume.yaml` | ミニPC のディスクを Kubernetes から使えるように定義 |
| `persistentvolumeclaim.yaml` | Pod がストレージを要求するための定義 |
| `deployment.yaml` | FreshRSS コンテナの起動方法を定義 |
| `service.yaml` | 外部からのアクセスを受け付ける設定（NodePort） |

---

## 💡 初心者向け Tips

### ArgoCD の自動同期について
このリポジトリの `application.yaml` では以下を有効化しています：
- **自動同期**: Git リポジトリの変更を検知して自動デプロイ
- **自己修復**: クラスタ側で手動変更されても Git の状態に自動復旧
- **自動削除**: Git から削除されたリソースをクラスタからも自動削除

つまり、**このリポジトリを更新してpushするだけで、自動的にミニPC上の FreshRSS が更新されます**！

### データの永続化について
FreshRSS のデータは `/var/lib/k8s-volumes/freshrss/` に保存されます。
Pod を削除・再作成してもデータは保持されます。

### ログの確認方法
```bash
kubectl logs -n freshrss -l app=freshrss
```

### トラブルシューティング
Pod が起動しない場合：
```bash
# Pod の状態を確認
kubectl describe pod -n freshrss -l app=freshrss

# イベントを確認
kubectl get events -n freshrss --sort-by='.lastTimestamp'
```

---

## 🎓 学習リソース

- [Kubernetes 公式ドキュメント](https://kubernetes.io/ja/docs/home/)
- [ArgoCD 公式ドキュメント](https://argo-cd.readthedocs.io/)
- [FreshRSS 公式サイト](https://freshrss.org/)