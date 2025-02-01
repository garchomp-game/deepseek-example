以下はPepperで動作する簡単なクイズゲームのChoregrapheプロジェクト構成とPythonコード例です。3つの質問に答える基本的な構造です。

### プロジェクト構成図
```
[Start] → [Initialize Quiz] → [Ask Question 1] → [Listen Answer]
          │                    ↑                ↓
          └──[Score Manager] ←─┴─[Check Answer] → [Feedback] → [Next Question]
```

### 各コンポーネントのPythonコード

#### 1. Initialize Quiz (Script Box)
```python
class MyClass(GeneratedClass):
    def __init__(self):
        GeneratedClass.__init__(self)
        self.questions = [
            {"q": "空が青い理由は？", "a": "レイリー散乱"},
            {"q": "世界一大きな国は？", "a": "ロシア"},
            {"q": "リンゴの木から落ちるもので有名な科学者は？", "a": "ニュートン"}
        ]
        self.current_question = 0
        self.score = 0

    def onLoad(self):
        pass

    def onUnload(self):
        pass

    def onInput_onStart(self):
        self.logger.info("Quiz Initialized")
        self.output_onSuccess()
```

#### 2. Ask Question (Speech Box + Script)
```python
class MyClass(GeneratedClass):
    def __init__(self):
        GeneratedClass.__init__(self)
        self.tts = ALProxy("ALTextToSpeech")

    def onLoad(self):
        pass

    def onUnload(self):
        pass

    def onInput_onStart(self, question):
        self.tts.setLanguage("Japanese")
        self.tts.say(question)
        self.output_onDone()
```

#### 3. Listen Answer (Speech Recognition)
```python
class MyClass(GeneratedClass):
    def __init__(self):
        GeneratedClass.__init__(self)
        self.asr = ALProxy("ALSpeechRecognition")
        self.memory = ALProxy("ALMemory")

    def onLoad(self):
        self.asr.setLanguage("Japanese")
        vocabulary = ["レイリー散乱", "ロシア", "ニュートン"]
        self.asr.setVocabulary(vocabulary, False)

    def onUnload(self):
        self.asr.unsubscribe("QuizASR")

    def onInput_onStart(self):
        self.asr.subscribe("QuizASR")
        self.memory.subscribeToEvent("WordRecognized", 
            self.getName(), "onWordRecognized")

    def onWordRecognized(self, key, value, message):
        if value and value[0] > 0.3:  # 信頼度30%以上
            self.logger.info("Detected: " + value[1])
            self.output_onDetected(value[1])
```

#### 4. Check Answer (Script Box)
```python
class MyClass(GeneratedClass):
    def __init__(self):
        GeneratedClass.__init__(self)

    def onLoad(self):
        pass

    def onUnload(self):
        pass

    def onInput_onStart(self, correct_answer, user_answer):
        if user_answer.strip().lower() == correct_answer.strip().lower():
            self.output_onCorrect(1)
        else:
            self.output_onIncorrect(0)
```

#### 5. Score Manager (Script Box)
```python
class MyClass(GeneratedClass):
    def __init__(self):
        GeneratedClass.__init__(self)
        self.total_score = 0

    def onLoad(self):
        pass

    def onUnload(self):
        pass

    def onInput_addScore(self, points):
        self.total_score += points
        self.logger.info(f"Current Score: {self.total_score}")

    def onInput_getFinalScore(self):
        self.output_onResult(self.total_score)
```

### 実行フロー
1. Initialize Quizで問題リストを初期化
2. Ask Questionで音声発話
3. Listen Answerで音声認識
4. Check Answerで正誤判定
5. Score Managerでスコア更新
6. 全問題終了後、最終結果を発表

### 拡張ポイント
- 問題データをJSONファイルで外部化
- 音声認識の代替入力（タッチセンサー）
- アニメーション連動
- 難易度別問題セット

```python
# 最終スコア発表コンポーネント例
class MyClass(GeneratedClass):
    def __init__(self):
        GeneratedClass.__init__(self)
        self.tts = ALProxy("ALTextToSpeech")

    def onInput_onStart(self, score):
        self.tts.say(f"クイズ終了！あなたのスコアは{score}点です。")
        if score == 3:
            self.tts.say("パーフェクトです！素晴らしい！")
        elif score >= 1:
            self.tts.say("よくできました！")
        else:
            self.tts.say("次はもっと頑張りましょう！")
        self.output_onDone()
```

Choregrapheのボックス間接続時には、変数の受け渡し（current_question, scoreなど）を適切に設定してください。音声認識の精度向上のため、下記設定を推奨します：

1. プロジェクト設定 → 音声認識
   - 言語: 日本語
   - 信頼度閾値: 0.3
   - タイムアウト: 5000ms

2. モーション設定
   - 質問時: 前傾姿勢
   - 正解時: 腕を上げる
   - 不正解時: 首を傾ける

実際のロボットで実行する場合は、音声認識の環境ノイズにも注意してください。学校やオフィス環境向けに調整可能なパラメータを備えています。
