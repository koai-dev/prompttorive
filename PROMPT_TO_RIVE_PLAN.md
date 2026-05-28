# 📋 Prompt to Rive - Kế Hoạch Phát Triển Website

## 🎯 Tổng Quan

**Prompt to Rive** là một ứng dụng web giúp người dùng tạo Rive scripts thông qua mô tả bằng ngôn ngữ tự nhiên (prompts). Ứng dụng sử dụng AI để chuyển đổi ý tưởng của người dùng thành code Luau script hoàn chỉnh cho Rive.

---

## 🏗️ Kiến Trúc Hệ Thống

### Frontend Stack
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS + shadcn/ui
- **State Management**: Zustand
- **Code Editor**: Monaco Editor (VS Code editor for web)
- **Preview**: Rive Web Runtime

### Backend Stack
- **Runtime**: Node.js với Next.js API Routes
- **AI Integration**: OpenAI API / Anthropic Claude API
- **Database**: PostgreSQL (Supabase) hoặc MongoDB
- **Authentication**: Clerk hoặc NextAuth.js
- **File Storage**: AWS S3 hoặc Supabase Storage

---

## 📁 Cấu Trúc Thư Mục

```
prompt-to-rive/
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (auth)/              # Authentication pages
│   │   │   ├── sign-in/
│   │   │   └── sign-up/
│   │   ├── (dashboard)/         # Dashboard pages
│   │   │   ├── dashboard/
│   │   │   ├── projects/
│   │   │   └── settings/
│   │   ├── editor/
│   │   │   └── [projectId]/
│   │   ├── api/                  # API routes
│   │   │   ├── ai/
│   │   │   │   ├── generate/
│   │   │   │   ├── refine/
│   │   │   │   └── explain/
│   │   │   ├── projects/
│   │   │   └── export/
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx
│   │
│   ├── components/
│   │   ├── ui/                   # shadcn/ui components
│   │   ├── editor/
│   │   │   ├── CodeEditor.tsx
│   │   │   ├── PreviewPanel.tsx
│   │   │   ├── PromptInput.tsx
│   │   │   └── ScriptSelector.tsx
│   │   ├── project/
│   │   │   ├── ProjectCard.tsx
│   │   │   └── ProjectList.tsx
│   │   └── shared/
│   │       ├── Header.tsx
│   │       └── Sidebar.tsx
│   │
│   ├── lib/
│   │   ├── ai/
│   │   │   ├── prompts.ts        # AI prompt templates
│   │   │   ├── generator.ts      # AI generation logic
│   │   │   └── validator.ts      # Code validation
│   │   ├── rive/
│   │   │   ├── templates.ts      # Script templates
│   │   │   ├── parser.ts         # Code parser
│   │   │   └── exporter.ts       # Export utilities
│   │   ├── db/
│   │   │   ├── schema.ts
│   │   │   └── queries.ts
│   │   └── utils.ts
│   │
│   ├── hooks/
│   │   ├── useAI.ts
│   │   ├── useProject.ts
│   │   └── useRive.ts
│   │
│   └── stores/
│       ├── projectStore.ts
│       └── editorStore.ts
│
├── public/
│   ├── templates/
│   └── examples/
│
├── .env.local
├── next.config.js
├── tailwind.config.ts
└── package.json
```

---

## 🔑 Tính Năng Chính

### 1. AI Script Generation
- **Natural Language Input**: Người dùng mô tả script muốn tạo
- **Protocol Selection**: Chọn loại script (Node, Layout, Converter, Path Effect, etc.)
- **Code Generation**: AI tạo code Luau hoàn chỉnh
- **Iterative Refinement**: Chat với AI để điều chỉnh code

### 2. Code Editor
- **Monaco Editor**: Syntax highlighting cho Luau
- **Auto-completion**: Gợi ý API Rive
- **Error Detection**: Phát hiện lỗi cú pháp
- **Code Formatting**: Tự động format code

### 3. Live Preview
- **Rive Runtime**: Preview script trong trình duyệt
- **Interactive Testing**: Test pointer events, inputs
- **Debug Panel**: Xem logs và errors

### 4. Project Management
- **Save Projects**: Lưu trữ projects trên cloud
- **Version History**: Theo dõi các phiên bản
- **Export Options**: Export .riv file hoặc code snippet
- **Share & Collaborate**: Chia sẻ projects

### 5. Template Library
- **Pre-built Templates**: Templates cho các use cases phổ biến
- **Community Examples**: Ví dụ từ cộng đồng
- **Quick Start**: Bắt đầu nhanh với templates

---

## 📝 Database Schema

```prisma
// schema.prisma

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  avatar        String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  projects      Project[]
  subscriptions Subscription?
}

model Project {
  id          String   @id @default(cuid())
  name        String
  description String?
  protocol    String   // Node, Layout, Converter, etc.
  code        String   @db.Text
  prompt      String   @db.Text
  rivFile     String?  // URL to .riv file
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  versions    Version[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([userId])
}

model Version {
  id        String   @id @default(cuid())
  version   Int
  code      String   @db.Text
  changes   String?
  projectId String
  project   Project  @relation(fields: [projectId], references: [id])
  createdAt DateTime @default(now())
  
  @@unique([projectId, version])
}

model Subscription {
  id           String     @id @default(cuid())
  userId       String     @unique
  user         User       @relation(fields: [userId], references: [id])
  plan         String     // free, pro, team
  credits      Int        @default(100)
  stripeId     String?
  createdAt    DateTime   @default(now())
  updatedAt    DateTime   @updatedAt
}
```

---

## 🤖 AI Prompts System

### System Prompt Template

```lua
Bạn là một chuyên gia về Rive scripting với ngôn ngữ Luau.
Nhiệm vụ của bạn là tạo scripts Rive hoàn chỉnh dựa trên mô tả của người dùng.

## Protocol Types:
- **Node Script**: Render shapes, images, text, artboards
- **Layout Script**: Position and arrange child elements
- **Converter Script**: Transform data before binding
- **Path Effect Script**: Custom effects on paths
- **Transition Condition Script**: State machine transitions
- **Listener Action Script**: React to state machine events
- **Test Script**: Unit tests for util scripts

## Required Structure:
Mỗi script phải có:
1. Type definition với inputs (nếu cần)
2. init() function - khởi tạo
3. Lifecycle functions phù hợp với protocol
4. Return statement với proper type

## Best Practices:
- Sử dụng PascalCase cho tên scripts và types
- Luôn cleanup listeners khi không cần
- Handle edge cases và errors
- Add comments giải thích code phức tạp
- Tối ưu performance

## Output Format:
Trả về code Luau hoàn chỉnh, không thêm giải thích trừ khi được yêu cầu.
```

### Example Prompts

```typescript
// prompts.ts

export const SCRIPT_TEMPLATES = {
  node: `Tạo một Node script {description}. 
Script cần có các inputs: {inputs}.
Yêu cầu đặc biệt: {requirements}`,

  converter: `Tạo một Converter script để {description}.
Input type: {inputType}
Output type: {outputType}
Logic: {logic}`,

  layout: `Tạo một Layout script để {description}.
Số lượng children: {childrenCount}
Layout style: {style} (flex, grid, custom)`,

  pathEffect: `Tạo một Path Effect script với hiệu ứng {effect}.
Parameters: {parameters}
Animation: {animation}`,
};
```

---

## 🎨 UI/UX Design

### Pages

#### 1. Landing Page (`/`)
- Hero section với demo interactive
- Features showcase
- Example gallery
- Pricing cards
- CTA buttons

#### 2. Dashboard (`/dashboard`)
- Recent projects
- Quick actions
- Usage statistics
- Template library

#### 3. Editor (`/editor/[projectId]`)
```
┌─────────────────────────────────────────────┐
│  Header: Project name, Save, Export, Share  │
├──────────────┬──────────────────────────────┤
│              │                              │
│  Prompt      │      Code Editor             │
│  Input       │      (Monaco)                │
│              │                              │
│  Settings    │                              │
│  - Protocol  │                              │
│  - Inputs    │                              │
│  - Options   │                              │
│              │                              │
├──────────────┴──────────────────────────────┤
│           Live Preview Panel                 │
│           (Rive Canvas)                      │
└─────────────────────────────────────────────┘
```

#### 4. Project Gallery (`/projects`)
- Grid view of all projects
- Search and filter
- Tags and categories
- Public/Private toggle

---

## 🔧 Rive Script Templates

### Node Script Template

```lua
-- type definition
type MyNode = {
  -- inputs here
  fillColor: Input<Color>,
  size: Input<number>
}

-- initialization
function init(self: MyNode, context: Context): boolean
  -- setup code
  return true
end

-- draw function
function draw(self: MyNode, renderer: Renderer, deltaTime: number): void
  -- rendering code
end

return function(): Node<MyNode>
  return {
    init = init,
    draw = draw,
    -- input defaults
    fillColor = late(),
    size = late(),
  }
end
```

### Converter Script Template

```lua
type MyConverter = {
  input: Input<number>
}

function convert(self: MyConverter, value: number): number
  -- transformation logic
  return value * 2
end

return function(): Converter<MyConverter>
  return {
    convert = convert,
    input = late(),
  }
end
```

### Layout Script Template

```lua
type MyLayout = {
  gap: Input<number>,
  padding: Input<number>
}

function init(self: MyLayout, context: Context): boolean
  return true
end

function layout(self: MyLayout, nodes: {NodeData}, width: number, height: number): {NodeData}
  -- layout logic
  return nodes
end

return function(): Layout<MyLayout>
  return {
    init = init,
    layout = layout,
    gap = late(),
    padding = late(),
  }
end
```

---

## 🚀 Roadmap

### Phase 1: MVP (4 weeks)
- [ ] Basic project setup (Next.js, TypeScript, Tailwind)
- [ ] Authentication (Clerk)
- [ ] Code editor with Monaco
- [ ] Basic AI generation (OpenAI)
- [ ] Simple preview panel
- [ ] Save/load projects

### Phase 2: Enhanced Features (3 weeks)
- [ ] All protocol types support
- [ ] Advanced AI prompts
- [ ] Version history
- [ ] Template library
- [ ] Export functionality
- [ ] Debug panel

### Phase 3: Polish & Scale (3 weeks)
- [ ] Performance optimization
- [ ] Collaboration features
- [ ] Community gallery
- [ ] Analytics
- [ ] Payment integration
- [ ] Documentation

### Phase 4: Launch (2 weeks)
- [ ] Beta testing
- [ ] Bug fixes
- [ ] Marketing materials
- [ ] Public launch

---

## 💰 Monetization

### Pricing Tiers

| Plan | Price | Credits/Month | Features |
|------|-------|---------------|----------|
| Free | $0 | 50 | Basic generation, 3 projects |
| Pro | $15/mo | 500 | Unlimited projects, priority generation, exports |
| Team | $49/mo | 2000 | Collaboration, shared templates, API access |
| Enterprise | Custom | Custom | Self-hosted, SLA, dedicated support |

---

## 📊 Technical Considerations

### Performance
- Code generation caching
- Lazy loading for Monaco editor
- Optimistic UI updates
- Debounced AI requests

### Security
- Input sanitization
- Rate limiting on AI endpoints
- Secure file storage
- XSS prevention in code preview

### Scalability
- Serverless architecture
- CDN for static assets
- Database indexing
- Queue system for heavy tasks

---

## 🔗 Useful Resources

### Rive Documentation
- [Scripting Overview](https://uat.rive.app/docs/scripting/getting-started)
- [API Reference](https://uat.rive.app/docs/scripting/api-reference)
- [Protocols](https://uat.rive.app/docs/scripting/protocols/overview)
- [Demos](https://uat.rive.app/docs/scripting/demos)

### Libraries
- [@rive-app/react-canvas](https://www.npmjs.com/package/@rive-app/react-canvas)
- [monaco-editor](https://www.npmjs.com/package/monaco-editor)
- [luau-lsp](https://github.com/JohnnyMorganz/luau-lsp) - For syntax highlighting

### AI Models
- OpenAI GPT-4 Turbo
- Anthropic Claude 3.5 Sonnet
- Fine-tuned models for code generation

---

## ✅ Next Steps

1. **Setup Development Environment**
   - Initialize Next.js project
   - Configure TypeScript and Tailwind
   - Setup authentication

2. **Build Core Components**
   - Code editor integration
   - AI generation service
   - Project management

3. **Implement Rive Integration**
   - Web runtime setup
   - Preview panel
   - Export functionality

4. **Testing & Iteration**
   - Unit tests
   - E2E tests
   - User feedback loop

---

*Document created based on official Rive scripting documentation*
*Last updated: 2024*
