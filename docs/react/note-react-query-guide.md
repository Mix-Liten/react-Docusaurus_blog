---
title: '[Note] React Query'
---

> 介紹 [React Query](https://tanstack.com/query/latest/) 基本使用


## Before Start

### 它解決了什麼問題

React Query 主要提供資料管理的功能，類似有狀態的 middleware，經由它發送非同步請求時會回傳 `loading`、`error`、`success`... 等等狀態，並且預設在請求成功時進行 cache、失敗時自動重新發送請求，另外還會定期在背景重發請求驗證資料是否過時，當然這些功能都可以因應需要進行客製。

以上種種功能可能你也有自行處理過，那就不難看出使用這個集大成套件的優點。
...

## Getting Started

### 前置
以下使用 Vite 方便快速示範，template 選擇 React + Typescript

```bash
npm create vite@latest
```

安裝套件，撰寫當下的版本是 v`4.33.0`
```bash
npm i @tanstack/react-query
```

新增模擬用的資料及非同步方法
```ts title="/src/entities/Todo.ts" showLineNumbers
export class Todo {
  id: number
  title: string
  completed: boolean

  constructor(id: number, title: string) {
    this.id = id
    this.title = title
    this.completed = false
  }
}
```
```ts title="/src/api/index.ts" showLineNumbers
import { Todo } from '../entities/Todo'

const todos = [
  { id: 0, title: 'Learn HTML', completed: false },
  { id: 1, title: 'Learn CSS', completed: false },
  { id: 2, title: 'Learn JavaScript', completed: false },
  { id: 3, title: 'Learn React', completed: false },
  { id: 4, title: 'Learn Next.js', completed: false },
]

export const fetchTodos = async (query = ''): Promise<Todo[]> => {
  await new Promise(resolve => setTimeout(resolve, 1000))

  const filteredTodos = todos.filter(todo => todo.title.toLowerCase().includes(query.toLowerCase()))

  return [...filteredTodos]
}

export const addTodo = async (todo: Pick<Todo, 'title'>): Promise<Todo> => {
  await new Promise(resolve => setTimeout(resolve, 1000))

  const newTodo = new Todo(todos.length, todo.title)

  todos.push(newTodo)

  return newTodo
}

export const toggleTodoStatus = async (todo: Pick<Todo, 'id'>): Promise<Todo> => {
  await new Promise(resolve => setTimeout(resolve, 1000))

  const thisTodo = todos[todo.id]

  thisTodo.completed = !thisTodo.completed

  return thisTodo
}

```

### 建立 QueryClient
```tsx title="/src/App.ts" showLineNumbers
// highlight-start
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { TodoList } from './components/TodoList'
const queryClient = new QueryClient()
// highlight-end

function App() {
  return (
// highlight-start
    <QueryClientProvider client={queryClient}>
      <TodoList />
    </QueryClientProvider>
// highlight-end
  )
}

export default App
```

### 使用 useQuery & useMutation
```tsx title="src/components/TodoList.tsx" showLineNumbers
import { useRef, useState } from 'react'
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query'
import { addTodo, fetchTodos } from '../api'
import { TodoCard } from './TodoCard'

export const TodoList = () => {
  const queryClient = useQueryClient()
  const [search, setSearch] = useState('')
  const [title, setTitle] = useState('')
  const searchRef = useRef<HTMLInputElement>(null)

  const {
    data: todos,
    isLoading,
    isFetching,
  } = useQuery({
    queryFn: () => fetchTodos(search),
    queryKey: ['todos', { search }],
    staleTime: Infinity,
    // cacheTime: 0,
  })

  const { mutateAsync: addTodoMutation, isLoading: addTodoLoading } = useMutation({
    mutationFn: addTodo,
    onSuccess: () => {
      queryClient.invalidateQueries(['todos'])
    },
  })

  return (
    <div>
      <div>
        <p>current search: {search}</p>
        <input type="text" ref={searchRef} />
        <button
          onClick={() => {
            if (searchRef.current) {
              const searchVal = searchRef.current.value
              setSearch(searchVal)
              searchRef.current.value = ''
            }
          }}
        >
          Search
        </button>
      </div>
      <div>
        <input type="text" value={title} onChange={e => setTitle(e.target.value)} />
        <button
          onClick={async () => {
            try {
              await addTodoMutation({ title })
              setTitle('')
            } catch (error) {
              console.error(error)
            }
          }}
        >
          Add Todo
        </button>
      </div>
      {isLoading ? (
        <div>loading todo...</div>
      ) : (
        <>{todos && todos.length ? todos.map(todo => <TodoCard key={todo.id} todo={todo} />) : <div>no data</div>}</>
      )}
      {!isLoading && isFetching && <div>check latest todos...</div>}
      {addTodoLoading && <div>push new todo...</div>}
    </div>
  )
}
```
```tsx title="src/components/TodoCard.tsx" showLineNumbers
import { useMutation } from '@tanstack/react-query'
import { toggleTodoStatus } from '../api'
import { Todo } from '../entities/Todo'

interface TodoCardProps {
  todo: Todo
}

export const TodoCard = ({ todo }: TodoCardProps) => {
  const { mutateAsync: toggleTodoStatusMutation, isLoading } = useMutation({
    mutationFn: toggleTodoStatus,
  })

  return (
    <div className="todo-card">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={async () => {
          try {
            await toggleTodoStatusMutation({ id: todo.id })
          } catch (error) {
            console.error(error)
          }
        }}
      />
      <span>{todo.title}</span>
      {isLoading && <span> - updating...</span>}
    </div>
  )
}
```

## Source Code
[Mix Liten - react-query_todo](https://github.com/Mix-Liten/practice_collect/tree/master/experiment/react-query_todo)

## Reference
- [PJCHENder - react query](https://pjchender.dev/npm/npm-react-query/)