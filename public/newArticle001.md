---
title: 'CursorでCI/CD開発するシフト管理アプリ'
tags:
  - 'Next.js'
  - 'Supabase'
  - 'Cursor'
  - 'AI開発'
  - 'ドキュメント駆動開発'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# CursorでCI/CD開発するコーヒーログアプリ

## 1. はじめに

個人開発やチーム開発において、効率的な開発フローを構築することは非常に重要です。特に最近では、AIを活用した開発支援ツールが注目を集めています。その中でも、Cursorは開発プロセス全体を効率化できる強力なIDE（統合開発環境）です。

私たちのチームでは、コーヒー愛好家向けの「コーヒーログアプリ」を開発するにあたり、Cursorを活用したAI支援開発を取り入れました。特に注目したのは「ドキュメント駆動開発」の手法です。これにより、要件定義から実装までの流れをスムーズにし、チーム内での知識共有も容易になりました。

本記事では、ドキュメント作成→プロンプト作成→実装という一連の流れを通じて、いかにして効率的な開発を実現したかを解説します。

## 2. 開発ワークフローの設計

私たちが採用した開発ワークフローは、次の3ステップで構成されています：

1. **ドキュメント作成**：要件定義、DB設計、画面設計などの文書を作成
2. **プロンプト作成**：ドキュメントに基づいたAI向けの適切なプロンプトを設計
3. **実装**：プロンプトをCursorで実行し、コードを生成・レビュー・修正

このワークフローの最大の利点は、各工程の成果物がすべてGitリポジトリで管理されることです。ドキュメント、プロンプト、コードがすべて一元管理されるため、チームメンバーは常に最新の情報にアクセスできます。

また、各工程には明確な役割があります：
- ドキュメント作成は「何を作るか」を明確にします
- プロンプト作成は「AIにどう伝えるか」を設計します
- 実装は「実際のコードとその品質」を担保します

## 3. ドキュメント作成フェーズ

ドキュメント作成は開発の土台となる重要なフェーズです。私たちは次のドキュメントを作成しました：

### 要件定義ドキュメント
```markdown
# コーヒーログアプリ要件定義

## 目的
- コーヒー愛好家が飲んだコーヒーの記録を簡単に残せるアプリを作成する
- 過去の記録を振り返り、好みの傾向を分析できるようにする

## 主要機能
1. ユーザー認証
2. コーヒー記録の作成・閲覧・編集・削除
3. 味わいのレーダーチャート表示
4. コーヒー記録の検索・フィルタリング
```

### データベース設計書
```markdown
# データベース設計

## Usersテーブル
- id (PK, UUID)
- email (STRING)
- name (STRING)
- created_at (TIMESTAMP)

## Coffee_recordsテーブル
- id (PK, UUID)
- user_id (FK → Users.id)
- coffee_name (STRING)
- shop_name (STRING)
- rating (INTEGER, 1-5)
- roast_level (ENUM)
- flavor_notes (JSONB)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
```

### 画面設計書
```markdown
# 画面設計

## 画面一覧
1. トップページ
2. ログイン/サインアップ画面
3. コーヒー記録一覧
4. コーヒー記録詳細
5. コーヒー記録作成/編集フォーム
```

これらのドキュメントはすべてGitリポジトリに`docs/`ディレクトリとして保存しました。

## 4. プロンプト作成フェーズ

ドキュメントが完成したら、次はAIに適切な指示を出すためのプロンプトを作成します。

### コンポーネント作成プロンプト
```markdown
# CoffeeForm コンポーネント作成

## 要件
- データベース設計書に基づいたフォームを作成
- Reactフック（useForm）を使用したバリデーション
- 送信時にはSupabaseへデータを保存
- 編集モードと新規作成モードの両方に対応

## 参照ドキュメント
- データベース設計書（Coffee_recordsテーブル）
- 画面設計書（コーヒー記録作成/編集フォーム）

## 技術スタック
- Next.js 14
- React Hook Form
- Tailwind CSS
- Supabase
```

### APIエンドポイントプロンプト
```markdown
# コーヒー記録API作成

## 要件
- RESTful APIパターンに従う
- 認証済みユーザーのみアクセス可能
- CRUD操作をサポート

## エンドポイント
- GET /api/coffee - 一覧取得
- GET /api/coffee/:id - 詳細取得
- POST /api/coffee - 新規作成
- PUT /api/coffee/:id - 更新
- DELETE /api/coffee/:id - 削除

## 参照
- データベース設計書
```

これらのプロンプトは`prompts/`ディレクトリに保存し、Gitで管理しました。

## 5. 実装フェーズ

プロンプトが準備できたら、Cursorを使って実際の実装に移ります。

### プロンプト実行
Cursorでプロンプトを実行するには、まずプロンプトファイルを開き、AIに指示を出します。例えば：

```
以下のプロンプトに基づいて、CoffeeFormコンポーネントを実装してください。
```

Cursorは指示に従って、必要なコードを生成します：

```tsx
// src/components/coffee/CoffeeForm.tsx
import { useState, useEffect } from 'react';
import { useForm } from 'react-hook-form';
import { createClient } from '@/lib/supabase';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';
import { Select } from '@/components/ui/Select';
import TasteRatingInput from './TasteRatingInput';

interface CoffeeFormProps {
  coffeeId?: string;
  onSuccess?: () => void;
}

export default function CoffeeForm({ coffeeId, onSuccess }: CoffeeFormProps) {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const isEditMode = !!coffeeId;
  
  const { register, handleSubmit, setValue, formState: { errors }, reset } = useForm({
    defaultValues: {
      coffee_name: '',
      shop_name: '',
      rating: 3,
      roast_level: 'medium',
      flavor_notes: {
        acidity: 3,
        sweetness: 3,
        body: 3,
        balance: 3,
        aftertaste: 3
      }
    }
  });

  // データ取得（編集モードの場合）
  useEffect(() => {
    if (isEditMode) {
      fetchCoffeeRecord();
    }
  }, [coffeeId]);

  async function fetchCoffeeRecord() {
    try {
      setIsLoading(true);
      const supabase = createClient();
      
      const { data, error } = await supabase
        .from('coffee_records')
        .select('*')
        .eq('id', coffeeId)
        .single();
        
      if (error) throw error;
      
      if (data) {
        // フォームに値をセット
        reset({
          coffee_name: data.coffee_name,
          shop_name: data.shop_name,
          rating: data.rating,
          roast_level: data.roast_level,
          flavor_notes: data.flavor_notes
        });
      }
    } catch (err) {
      console.error('Error fetching coffee record:', err);
      setError('コーヒー記録の取得に失敗しました');
    } finally {
      setIsLoading(false);
    }
  }

  // フォーム送信処理
  async function onSubmit(formData) {
    try {
      setIsLoading(true);
      setError(null);
      const supabase = createClient();
      
      if (isEditMode) {
        // 更新処理
        const { error } = await supabase
          .from('coffee_records')
          .update(formData)
          .eq('id', coffeeId);
          
        if (error) throw error;
      } else {
        // 新規作成処理
        const { error } = await supabase
          .from('coffee_records')
          .insert([formData]);
          
        if (error) throw error;
      }
      
      // 成功時の処理
      reset();
      if (onSuccess) onSuccess();
      
    } catch (err) {
      console.error('Error saving coffee record:', err);
      setError('保存に失敗しました。もう一度お試しください。');
    } finally {
      setIsLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      {error && (
        <div className="bg-red-50 p-4 rounded-md border border-red-200 text-red-700">
          {error}
        </div>
      )}
      
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label className="block text-sm font-medium mb-1">コーヒー名</label>
          <Input
            {...register('coffee_name', { required: 'コーヒー名は必須です' })}
            placeholder="例: エチオピア イルガチェフェ"
            error={errors.coffee_name?.message}
          />
        </div>
        
        <div>
          <label className="block text-sm font-medium mb-1">お店の名前</label>
          <Input
            {...register('shop_name', { required: 'お店の名前は必須です' })}
            placeholder="例: スターバックス"
            error={errors.shop_name?.message}
          />
        </div>
      </div>
      
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <label className="block text-sm font-medium mb-1">総合評価</label>
          <Select
            {...register('rating')}
            options={[
              { value: 1, label: '★☆☆☆☆' },
              { value: 2, label: '★★☆☆☆' },
              { value: 3, label: '★★★☆☆' },
              { value: 4, label: '★★★★☆' },
              { value: 5, label: '★★★★★' }
            ]}
          />
        </div>
        
        <div>
          <label className="block text-sm font-medium mb-1">焙煎度合い</label>
          <Select
            {...register('roast_level')}
            options={[
              { value: 'light', label: '浅煎り' },
              { value: 'medium', label: '中煎り' },
              { value: 'medium-dark', label: '中深煎り' },
              { value: 'dark', label: '深煎り' }
            ]}
          />
        </div>
      </div>
      
      <div>
        <label className="block text-sm font-medium mb-3">味わい評価</label>
        <TasteRatingInput
          name="flavor_notes"
          register={register}
          setValue={setValue}
        />
      </div>
      
      <div className="flex justify-end space-x-3">
        <Button
          type="button"
          variant="outline"
          onClick={() => reset()}
          disabled={isLoading}
        >
          リセット
        </Button>
        <Button
          type="submit"
          disabled={isLoading}
        >
          {isLoading ? '保存中...' : isEditMode ? '更新する' : '記録する'}
        </Button>
      </div>
    </form>
  );
}
```

### コードレビューと修正

生成されたコードを確認し、必要に応じて修正します。例えば、型定義が不足している場合は補完したり、セキュリティ上の懸念がある部分は修正したりします。

コードレビューのポイント：
- 要件に沿った実装になっているか
- 型定義が適切か
- エラーハンドリングが十分か
- パフォーマンスに問題はないか

## 6. Gitを活用したドキュメント共有

私たちのプロジェクトは次のような構造でGitリポジトリに管理しています：

```
coffee-log-app/
├── .github/            # GitHub Actions設定
├── docs/               # プロジェクトドキュメント
│   ├── requirements/   # 要件定義
│   ├── database/       # DB設計
│   ├── ui/             # UI設計
│   └── api/            # API仕様
├── prompts/            # AIプロンプト集
│   ├── components/     # コンポーネント用プロンプト
│   ├── api/            # API用プロンプト
│   └── tests/          # テスト用プロンプト
├── src/                # ソースコード
│   ├── app/            # Next.js App Router
│   ├── components/     # Reactコンポーネント
│   ├── lib/            # ユーティリティ関数
│   └── ...
└── README.md           # プロジェクト概要
```

この構造により、以下のメリットが得られました：

1. **知識の一元管理**: ドキュメントとコードが同じリポジトリにあるため、常に最新の情報にアクセスできます
2. **変更履歴の追跡**: Gitの履歴機能により、ドキュメントの変更履歴を追跡できます
3. **レビュープロセスの統合**: コードレビューと同時にドキュメントもレビューできます
4. **新メンバーのオンボーディング**: 新しいチームメンバーがプロジェクトの全体像を理解しやすくなります

また、Pull Requestのテンプレートを作成し、ドキュメント、プロンプト、コードの関連性を明示することで、レビュープロセスを効率化しました。

## 7. 技術スタックと構成

私たちのプロジェクトでは、以下の技術スタックを採用しました：

- **フロントエンド**: Next.js 14 (App Router)
- **スタイリング**: Tailwind CSS
- **状態管理**: React Hook Form + Context API
- **バックエンド**: Supabase (認証、データベース)
- **デプロイ**: Vercel
- **CI/CD**: GitHub Actions

この構成を選んだ主な理由は：

1. **Next.js**: サーバーコンポーネントとクライアントコンポーネントを適切に使い分けることで、パフォーマンスを最適化できる
2. **Supabase**: バックエンド構築の手間を省き、認証やデータベースをすぐに利用できる
3. **Tailwind CSS**: コンポーネント間で一貫したデザインを実現しやすい
4. **TypeScript**: 型安全性により開発時のエラーを減らせる

CI/CDパイプラインは、GitHub Actionsを使って自動化しました：

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run lint
      - run: npm run test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: vercel/actions/cli@master
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID}}
          vercel-project-id: ${{ secrets.PROJECT_ID}}
          vercel-args: '--prod'
```

## 8. プロジェクト管理とチーム連携

ドキュメント駆動開発を効果的に進めるため、以下のプロジェクト管理手法を採用しました：

### タスク管理

GitHub Issuesを使用して、各機能ごとにタスクを作成しました。各Issueには次の情報を含めました：

- 関連するドキュメントへのリンク
- 使用するプロンプトへのリンク
- 実装の優先度と期限
- 担当者

### 進捗の可視化

GitHub Projectsのカンバンボードを使って、タスクの進捗を可視化しました：

- Backlog: 未着手のタスク
- Documentation: ドキュメント作成中
- Prompt Design: プロンプト設計中
- Implementation: 実装中
- Review: レビュー中
- Done: 完了

### チームコミュニケーション

週次ミーティングでは、ドキュメントの変更点やプロンプトの改善点について議論しました。また、Slackでは次のチャンネルを設置しました：

- `#docs-updates`: ドキュメントの更新通知
- `#prompt-sharing`: 効果的だったプロンプトの共有
- `#code-reviews`: コードレビューの議論

## 9. 実際のユースケース

実際のユースケースとして、「コーヒーの味わい評価機能」の開発プロセスを紹介します。

### ステップ1: ドキュメント作成

```markdown
# 味わい評価機能仕様書

## 概要
コーヒーの味わいを5つの要素（酸味、甘味、ボディ、バランス、後味）で
5段階評価できる機能を実装する。

## UI要件
- スライダー形式で各要素を1-5で評価
- 入力結果をレーダーチャートで視覚化
- モバイル対応のUI

## データ構造
flavor_notes: {
  acidity: number (1-5),
  sweetness: number (1-5),
  body: number (1-5),
  balance: number (1-5),
  aftertaste: number (1-5)
}
```

### ステップ2: プロンプト作成

```markdown
# 味わい評価コンポーネント作成プロンプト

以下の仕様に基づいた、コーヒーの味わい評価用のコンポーネントを作成してください：

1. TasteRatingInput コンポーネント
   - 5つの味わい要素（酸味、甘味、ボディ、バランス、後味）を評価
   - 各要素は1-5のスライダーで評価
   - React Hook Formと統合
   - モバイルフレンドリーなUI

2. TasteRadarChart コンポーネント
   - 入力された味わい評価をレーダーチャートで表示
   - Chart.jsとreact-chartjs-2を使用
   - ダークモード対応

技術スタック:
- React
- TypeScript
- Tailwind CSS
- React Hook Form
- Chart.js / react-chartjs-2
```

### ステップ3: 実装

Cursorを使ってプロンプトを実行し、コンポーネントを実装しました。生成されたコードを適宜修正して最終的な実装が完成しました。

```tsx
// TasteRatingInput.tsx（一部抜粋）
export default function TasteRatingInput({ 
  name, 
  register, 
  setValue,
  defaultValues = {
    acidity: 3,
    sweetness: 3,
    body: 3,
    balance: 3,
    aftertaste: 3
  }
}) {
  // 実装コード
}

// TasteRadarChart.tsx（一部抜粋）
export default function TasteRadarChart({ flavorNotes }) {
  const data = {
    labels: ['酸味', '甘み', 'ボディ', 'バランス', '後味'],
    datasets: [
      {
        label: '味わい評価',
        data: [
          flavorNotes.acidity,
          flavorNotes.sweetness,
          flavorNotes.body,
          flavorNotes.balance,
          flavorNotes.aftertaste
        ],
        backgroundColor: 'rgba(75, 192, 192, 0.2)',
        borderColor: 'rgba(75, 192, 192, 1)',
        borderWidth: 2,
      },
    ],
  };
  
  // 実装コード
}
```

### 成功したアプローチ

このユースケースで特に効果的だったのは：

1. **明確な仕様書**: 味わい評価の各要素や評価方法が明確に定義されていた
2. **的確なプロンプト**: 必要な技術スタックや統合方法を具体的に指示できた
3. **モジュール分割**: 入力部分とグラフ表示部分を別コンポーネントに分けたことで、再利用性が高まった

## 10. 学びと今後の展望

### 教訓

1. **ドキュメント先行の価値**: 実装前にドキュメントをしっかり作ることで、AI生成コードの質が大幅に向上
2. **プロンプト設計の重要性**: 具体的かつ構造化されたプロンプトが良いコードを生み出す
3. **レビュープロセスの必要性**: AI生成コードも人間のレビューが必須
4. **Git管理の利点**: ドキュメント、プロンプト、コードを一元管理することの効率性

### 改善点

今後のプロジェクトでは以下の点を改善したいと考えています：

1. **プロンプトテンプレート**: よく使うプロンプトパターンをテンプレート化
2. **ドキュメント自動生成**: コードからドキュメントを自動生成する仕組みの導入
3. **テスト自動化**: プロンプトからテストコードも自動生成する仕組み

### 今後の展望

AI支援開発はまだ始まったばかりですが、特にドキュメント駆動の手法との相性が良いことがわかりました。今後はこの手法をさらに洗練させ、より大規模なプロジェクトでも適用できるよう改善していきたいと考えています。

また、ドキュメントとコードの一貫性を自動でチェックする仕組みや、ドキュメント変更からコード変更を自動提案するシステムなど、さらなる自動化も検討していきます。

コーヒーログアプリ自体も継続的に機能拡張を行い、より多くのユーザーに使っていただけるよう改善していく予定です。

---

Cursorを活用したドキュメント駆動開発は、私たちのチームの開発効率を大幅に向上させました。特に、ドキュメント、プロンプト、コードをすべてGitで管理することで、チーム内の知識共有とコラボレーションがスムーズになりました。

この記事が、AI支援開発やドキュメント駆動開発に興味のある方々の参考になれば幸いです。
