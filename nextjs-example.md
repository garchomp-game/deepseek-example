Next.js 14（App Router）でリアルタイムチャットアプリを構築するサンプルです。最小限のコードでモダンなスタックを活用した実装です。

```bash
npm install @mui/material @emotion/styled @mui/icons-material @tailwindcss/typography pusher-js
```

### 1. バックエンド（API Route） `/app/api/chat/route.ts`
```typescript
import { NextResponse } from 'next/server';
import Pusher from 'pusher';

const pusher = new Pusher({
  appId: process.env.PUSHER_APP_ID!,
  key: process.env.NEXT_PUBLIC_PUSHER_KEY!,
  secret: process.env.PUSHER_SECRET!,
  cluster: 'ap3',
  useTLS: true
});

export async function POST(req: Request) {
  const { message, user } = await req.json();
  
  pusher.trigger('chat-channel', 'message-event', {
    user,
    message,
    timestamp: new Date().toISOString()
  });

  return NextResponse.json({ status: 'sent' });
}
```

### 2. フロントエンド `/app/chat/page.tsx`
```typescript
'use client';
import { useState, useEffect } from 'react';
import Pusher from 'pusher-js';
import { 
  Container, 
  TextField, 
  Button, 
  List, 
  ListItem, 
  ListItemText, 
  Typography 
} from '@mui/material';
import SendIcon from '@mui/icons-material/Send';

type Message = {
  user: string;
  message: string;
  timestamp: string;
};

export default function ChatApp() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState('');
  const [user] = useState(`User_${Math.floor(Math.random() * 1000)}`);

  useEffect(() => {
    const pusher = new Pusher(process.env.NEXT_PUBLIC_PUSHER_KEY!, {
      cluster: 'ap3'
    });

    const channel = pusher.subscribe('chat-channel');
    channel.bind('message-event', (data: Message) => {
      setMessages(prev => [...prev, data]);
    });

    return () => {
      pusher.unsubscribe('chat-channel');
    };
  }, []);

  const sendMessage = async () => {
    if (!input.trim()) return;
    
    await fetch('/api/chat', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ message: input, user }),
    });

    setInput('');
  };

  return (
    <Container maxWidth="md" className="py-8">
      <Typography variant="h4" gutterBottom>
        Realtime Chat
      </Typography>
      
      <List className="h-96 overflow-auto mb-4 bg-gray-50 rounded-lg">
        {messages.map((msg, index) => (
          <ListItem key={index} className="!items-start">
            <ListItemText
              primary={
                <div className="flex gap-2">
                  <span className="font-semibold">{msg.user}</span>
                  <span className="text-gray-500 text-sm">
                    {new Date(msg.timestamp).toLocaleTimeString()}
                  </span>
                </div>
              }
              secondary={msg.message}
            />
          </ListItem>
        ))}
      </List>

      <div className="flex gap-2">
        <TextField
          fullWidth
          variant="outlined"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
          placeholder="Type a message..."
        />
        <Button
          variant="contained"
          endIcon={<SendIcon />}
          onClick={sendMessage}
        >
          Send
        </Button>
      </div>
    </Container>
  );
}
```

### 3. 設定ファイル

`.env.local`:
```env
NEXT_PUBLIC_PUSHER_KEY=your_pusher_key
PUSHER_APP_ID=your_app_id
PUSHER_SECRET=your_secret
```

`tailwind.config.js`:
```javascript
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [require('@tailwindcss/typography')],
}
```

### 特徴的な技術選択
1. **Pusher**: リアルタイム通信を簡潔に実現
2. **MUI + Tailwind**: スタイリングの一貫性と柔軟性を両立
3. **App Router**: Next.js 14の最新機能を活用
4. **TypeScript**: 型安全な実装
5. **サーバーレスアーキテクチャ**: スケーラブルな構成

### 動作フロー
1. ユーザーがメッセージ送信
2. API RouteがPusherでイベント発行
3. 全クライアントでリアルタイムにメッセージ受信
4. メッセージリストが自動更新

この構成では、以下のような拡張が可能です：
- ユーザー認証の追加（NextAuth.js）
- メッセージ永続化（Supabase/MySQL）
- 画像送信機能（Cloudinary連携）
- メンション機能の実装

必要に応じて最適なクラウドサービスを選択可能な、モダンな基盤構成になっています。
