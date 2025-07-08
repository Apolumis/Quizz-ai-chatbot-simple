# Quizz-ai-chatbot-simple
Using api to create a quizz learning chatbot and we can ask question
# ⚠️ I USE AN OLDER VERSION OF @google/generative-ai so alot might be depricated, so you should usechatgpt to make it new according to the latest
## Overview (logic flow):
```bash
project-root/
├── server.ts        ← backend route (replaces route.ts)
├── .env
├── src/
│   ├── App.tsx
│   ├── main.tsx
│   └── components/
│       └── QuizCard.tsx
├── index.html
├── vite.config.ts
└── ...
```


## READ DETAILS.md (read - optional, but highly recommended)

## Project Setup
### AI SDK (by Vercel)
<br/>
The AI SDK is the TypeScript toolkit designed to help developers build AI-powered applications and agents with React, Next.js, Vue, Svelte, Node.js

*we will be using react.js vite for typescript, tailwindcss in this project*
```bash
npm create vite@latest
```
*then, do tailwind css*
<a>https://tailwindcss.com/docs/installation/using-vite</a>

*then, install this package*
```bash
npm i ai # for ai sdk
npm install lucide-react # for icons
npm i remark-gfm
npm i react-markdown # for bold characters
npm install express cors body-parser dotenv @google/generative-ai
```
then do this, but you can visit ai-sdk.dev to see more, but we will be working with gemini api in this project
```bash
npm install @ai-sdk/google
# or
pnpm add @ai-sdk/google
# or
yarn add @ai-sdk/google
```

## create a file: .env.local
create an env file
```bash
GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
```

## CODE:
### App.tsx
```bash
// App.tsx
import React, { useEffect, useRef, useState } from 'react';
import { Send, BrainCircuit, Bot } from 'lucide-react';
import QuizCard from './components/QuizCard';
import ReactMarkdown from 'react-markdown';

interface QuizItem {
  question: string;
  options: string[];
  answer: string;
  explanation: string;
}

const App: React.FC = () => {
  const [topic, setTopic] = useState('');
  const [difficulty, setDifficulty] = useState('easy');
  const [quiz, setQuiz] = useState<QuizItem[]>([]);
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState<{ role: 'user' | 'ai' | 'quiz'; content: any }[]>([]);
  const bottomRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  const generateQuiz = async () => {
    const res = await fetch('http://localhost:3000/api/quizz', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ topic, difficulty, numQuestions: 3 })
    });
    const data = await res.json();
    setQuiz(data.quizzes);
    setMessages((prev) => [...prev, { role: 'quiz', content: data.quizzes }]);
  };

  const sendMessage = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!input.trim()) return;
    setMessages((prev) => [...prev, { role: 'user', content: input }]);
    setInput('');

    const res = await fetch('http://localhost:3000/api/quizzchat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: input, quizContext: quiz })
    });

    const data = await res.json();
    setMessages((prev) => [...prev, { role: 'ai', content: data.response.join(' ') }]);
  };

  return (
    <main className="flex flex-col h-screen max-w-2xl mx-auto">
      <header className="px-4 py-3">
        <h1 className="text-3xl font-bold text-center text-green-700 flex items-center justify-center">
          <Bot className="w-8 h-8 mr-2" /> AI Quiz & Chat
        </h1>
      </header>

      <div className="flex-1 overflow-y-auto px-4 space-y-6 pb-32">
        <div className="space-y-3 grid grid-cols-3 gap-1">
          <div className='col-span-2 space-y-2'>
            <input
              type="text"
              value={topic}
              onChange={(e) => setTopic(e.target.value)}
              placeholder="Enter a topic (e.g., JavaScript)"
              className="w-full p-2 border border-gray-300 rounded"
            />
            <select
              value={difficulty}
              onChange={(e) => setDifficulty(e.target.value)}
              className="w-full p-2 border border-gray-300 rounded"
            >
              <option value="easy">Easy</option>
              <option value="medium">Medium</option>
              <option value="hard">Hard</option>
            </select>
          </div>
          <button
            onClick={generateQuiz}
            className="flex items-center justify-center h-[88px] bg-green-600 text-white px-4 rounded-lg shadow-md text-center hover:bg-green-700"
          >
            <BrainCircuit className="w-5 h-5 mr-1" /> Generate Quizzes
          </button>
        </div>

        {messages.length > 0 && (
          <div className="space-y-2">
            {messages.map((msg, idx) => {
              if (msg.role === 'quiz' && Array.isArray(msg.content)) {
                return (
                  <div key={idx} className="space-y-4">
                    {msg.content.map((q: QuizItem, qIdx: number) => (
                      <QuizCard key={qIdx} question={q} />
                    ))}
                  </div>
                );
              }
              return (
                <div key={idx} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                  <div className={`px-4 py-2 max-w-xs rounded-lg shadow ${msg.role === 'user' ? 'bg-blue-500 text-white' : 'bg-gray-100 text-gray-800'}`}>
                    {msg.role === 'ai' ? (
                      <ReactMarkdown>{msg.content}</ReactMarkdown>
                    ) : (
                      msg.content
                    )}
                  </div>
                </div>
              );
            })}
            <div ref={bottomRef} />
          </div>
        )}
      </div>

      <form onSubmit={sendMessage} className="bottom-0 left-0 right-0 w-full mx-auto bg-white border-t border-gray-300 p-4 flex items-center gap-2">
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          className="flex-1 p-2 rounded-lg border border-gray-300 focus:outline-none focus:ring-2 focus:ring-green-500"
          placeholder="Ask the AI something..."
        />
        <button type="submit" className="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded">
          <Send className="w-4 h-4" />
        </button>
      </form>
    </main>
  );
};

export default App;
```

### components/QuizCard.tsx
```bash
// components/QuizCard.tsx
import React, { useState } from 'react';

interface QuizItem {
  question: string;
  options: string[];
  answer: string;
  explanation: string;
}

const QuizCard: React.FC<{ question: QuizItem }> = ({ question }) => {
  const [flipped, setFlipped] = useState(false);

  return (
    <div
      className="relative w-full h-56 cursor-pointer"
      style={{ perspective: '1000px' }}
      onClick={() => setFlipped((prev) => !prev)}
    >
      <div
        className={`relative w-full h-full duration-500 transition-transform ${flipped ? 'rotate-y-180' : ''}`}
        style={{ transformStyle: 'preserve-3d' }}
      >
        <div
          className="absolute w-full h-full bg-white border p-4 rounded-lg shadow z-10"
          style={{ backfaceVisibility: 'hidden' }}
        >
          <h3 className="font-semibold text-lg">{question.question}</h3>
          <ul className="list-disc ml-6 mt-2 text-sm">
            {question.options.map((opt, i) => (
              <li key={i}>{opt}</li>
            ))}
          </ul>
          <p className="text-sm text-gray-400 mt-4">Click to reveal answer ➡</p>
        </div>

        <div
          className="absolute w-full h-full bg-green-100 border p-4 rounded-lg shadow rotate-y-180"
          style={{ backfaceVisibility: 'hidden', transform: 'rotateY(180deg)' }}
        >
          <p className="font-semibold">✅ Answer: {question.answer}</p>
          <p className="mt-2 text-sm text-gray-800">{question.explanation}</p>
          <p className="text-sm text-gray-400 mt-4">Click again to flip back 🔄</p>
        </div>
      </div>
    </div>
  );
};

export default QuizCard;
```

### Using express.js as backend: server.ts (run the server: *npx tsx server.ts*)
```bash
import express from 'express';
import cors from 'cors';
import bodyParser from 'body-parser';
import dotenv from 'dotenv';
import { GoogleGenerativeAI } from '@google/generative-ai';

dotenv.config();

const app = express();
app.use(cors());
app.use(bodyParser.json());

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);

// === /api/quizz ===
app.post('/api/quizz', async (req, res) => {
  const { topic, difficulty = 'medium', numQuestions = 5 } = req.body;

  const model = genAI.getGenerativeModel({ model: 'gemini-1.5-flash' });

  const prompt = `
You are a quiz generator AI.
Generate ${numQuestions} multiple-choice quiz questions about the topic: "${topic}".
The difficulty level is "${difficulty}".
Each question should have 4 options (A, B, C, D), one correct answer, and a short explanation.
Return as JSON array like:
[
  {
    "question": "...",
    "options": ["A", "B", "C", "D"],
    "answer": "B",
    "explanation": "..."
  },
  ...
]
`;

  try {
    const result = await model.generateContent(prompt);
    const text = result.response.text();

    const start = text.indexOf('[');
    const end = text.lastIndexOf(']') + 1;
    const json = text.slice(start, end);

    const quizzes = JSON.parse(json);
    res.json({ quizzes });
  } catch (err) {
    console.error('Quiz generation error:', err);
    res.status(500).json({ error: 'Failed to generate quiz.' });
  }
});

// === /api/quizzchat ===
app.post('/api/quizzchat', async (req, res) => {
  const { message, quizContext } = req.body;

  const context = quizContext.map((q: any, i: number) => {
    return `Q${i + 1}: ${q.question}
Options: ${q.options.join(', ')}
Answer: ${q.answer}
Explanation: ${q.explanation}`;
  }).join('\n\n');

  const prompt = `
You are a helpful tutor AI.
Here is the quiz context:\n${context}

Now the user asked: "${message}"

Respond with a helpful, clear explanation or answer based on the quiz above.
`;

  try {
    const model = genAI.getGenerativeModel({ model: 'gemini-1.5-flash' });
    const result = await model.generateContent(prompt);
    const text = result.response.text();

    res.json({ response: text.split('\n') });
  } catch (err) {
    console.error('Chat error:', err);
    res.status(500).json({ error: 'Failed to respond to user.' });
  }
});

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```
