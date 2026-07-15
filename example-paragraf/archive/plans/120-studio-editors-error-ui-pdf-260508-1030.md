---
type: plan
workId: 120
status: done
created: 260508-1030
updated: 260509-0050
packages: [studio]
target-version: —
---

# Plan: 120 — Studio — Editors, Error Handling & PDF Compile

## Objective

Complete the studio PoC. Adds: CodeMirror 6 JSON and XML display panels (read-only with
syntax highlighting and error decorations), the StatusBar, the stale-preview banner, and
the "Compile!" button that triggers a full PDF compile and displays the result.

**PoC completion gate:** after this plan all five acceptance criteria from the brainstorm
are demonstrable:
1. Open project folder → editors load.
2. Edit `template.json` → reflow in preview within ~300ms.
3. Edit `content.xml` → reflow in preview.
4. Click "Compile!" → PDF written to `output/`, displayed in preview panel.
5. Introduce syntax error in `template.json` → red underline, status bar message, stale
   banner (preview does not go blank).

## Architecture

| Package | Layer | Role in this work |
|---|---|---|
| `studio` | App | TemplatePanel (CodeMirror JSON), ContentPanel (CodeMirror XML), StatusBar, StaleBanner, PDF display |

## Constraints

- Editors are **read-only** in PoC v1. CodeMirror's `EditorView` is created with
  `EditorState.readOnly.of(true)`. The files on disk are the source of truth; studio
  only displays them.
- Error decorations: `ValidationError[]` received via IPC include `{ line: number, message: string }`.
  A CodeMirror `StateField` + `Decoration.mark` adds a red underline on the reported line.
  The error list is passed as a React prop and applied via a `useEffect` that dispatches a
  CodeMirror transaction.
- "Stale preview" banner: shown when `lastError !== null`. Amber bar at the top of the
  PreviewPanel. The last successful SVG render remains visible below the banner — preview
  does not go blank.
- StatusBar displays: validation state emoji + message (first error message if any),
  page count, shaping engine from last compile result, studio version from `package.json`.
- PDF display: after a successful `triggerPdfCompile()`, the main process writes the PDF
  to `project/output/[projectName]-[timestamp].pdf` and sends the absolute path back via
  IPC. The renderer uses `<webview>` with the `file://` path (Electron built-in PDF viewer)
  or opens it in the system viewer via `shell.openPath`. For PoC, system viewer is
  sufficient — no embedded PDF.js.
- `triggerPdfCompile()` in the compile worker reuses the current session but calls
  `compile({ output: 'pdf', ... })`. The main process writes the result Buffer to disk
  and sends the file path to the renderer.
- Tests: React Testing Library + vitest (happy-dom). CodeMirror tests use a custom render
  helper that creates a minimal CM6 EditorView in jsdom.
- `@codemirror/lang-json` and `@codemirror/lang-xml` are runtime dependencies of `studio`.
  `@codemirror/view`, `@codemirror/state` are peer dependencies of the lang packages and
  must also be added.

## Cross-Package Contract

`CompileWorkerResult` (Plan 118 / Plan 119) is extended to include
`errors: ValidationError[]` in the svg/error response, so the renderer can decorate
both editors simultaneously from a single IPC event. `ValidationError` carries a `file`
discriminant (`'template' | 'content'`) so each panel decorates only its own errors.

## Tasks

| # | Task | Primitive(s) | Status | Notes |
|---|---|---|---|---|
| 1 | UPDATE studio/package.json — add @codemirror/view, @codemirror/state, @codemirror/lang-json, @codemirror/lang-xml runtime deps | UPDATE | done | |
| 2 | WRITE studio/src/codemirror/error-decorations.ts — CodeMirror StateField + ViewPlugin that applies red underline Decoration.mark at reported line numbers | WRITE | done | Takes ValidationError[] as input; pure CM6 extension |
| 3 | WRITE studio/src/panels/TemplatePanel.tsx — CodeMirror 6 editor in read-only mode, JSON language, error-decorations extension wired to errors prop | WRITE | done | File content passed as string prop from App state |
| 4 | WRITE studio/src/panels/ContentPanel.tsx — CodeMirror 6 editor in read-only mode, XML language, error-decorations extension wired to errors prop | WRITE | done | |
| 5 | WRITE studio/src/components/StatusBar.tsx — shows validation state, page count, shaping engine, version | WRITE | done | All data from props, no IPC in component |
| 6 | UPDATE studio/src/panels/PreviewPanel.tsx — add StaleBanner overlay when lastError != null | UPDATE | done | Banner shows above SVG without removing it |
| 7 | WRITE studio/src/components/StaleBanner.tsx — amber bar with message "Preview stale — template has errors" | WRITE | done | |
| 8 | WRITE studio/src/components/CompileButton.tsx — "Compile!" button; disabled while isCompiling or hasErrors; calls window.studio.triggerPdfCompile() on click | WRITE | done | Positioned in preview header area |
| 9 | UPDATE studio/electron.main.ts — add ipcMain.handle('triggerPdfCompile'): runs compile({ output: 'pdf' }) in worker, writes Buffer to output/, sends file path back | UPDATE | done | Atomic write; timestamp in filename |
| 10 | UPDATE studio/src/ipc-types.ts — add triggerPdfCompile(): Promise<{ filePath: string }> to IpcApi | UPDATE | done | |
| 11 | UPDATE studio/src/App.tsx — wire TemplatePanel and ContentPanel with file content from IPC; wire StatusBar; split ValidationError[] by file discriminant; wire CompileButton | UPDATE | done | Replace skeleton divs from Plan 118 |
| 12 | UPDATE studio/src/hooks/useCompileWorker.ts — add pdfFilePath state; handle { type: 'pdf', filePath } result | UPDATE | done | |
| 13 | WRITE studio/tests/panels/TemplatePanel.test.tsx — RT1–RT4 | WRITE | done | |
| 14 | WRITE studio/tests/panels/ContentPanel.test.tsx — RT5–RT7 | WRITE | done | |
| 15 | WRITE studio/tests/components/StatusBar.test.tsx — RT8–RT11 | WRITE | done | |
| 16 | WRITE studio/tests/components/CompileButton.test.tsx — RT12–RT14 | WRITE | done | |
| 17 | WRITE studio/tests/codemirror/error-decorations.test.ts — RT15–RT17 | WRITE | done | |
| 18 | VERIFY npm run test --workspace=studio | VERIFY | done | 75/75 pass |
| 19 | VERIFY PoC criteria 4–5 manually | VERIFY | done | Compile button → PDF; syntax error → red underline |

## Required Tests

### TemplatePanel.test.tsx

| # | Description | Type | Status |
|---|---|---|---|
| RT1 | Renders CodeMirror editor with the provided JSON string content | component | done |
| RT2 | Editor is read-only — programmatic setContent does not mutate document | component | done |
| RT3 | No `errors` prop → no decoration marks in the editor | component | done |
| RT4 | `errors` prop with one entry → error-decorations extension applied (CM6 StateField has decoration) | component | done |

### ContentPanel.test.tsx

| # | Description | Type | Status |
|---|---|---|---|
| RT5 | Renders CodeMirror editor with the provided XML string content | component | done |
| RT6 | Editor is read-only | component | done |
| RT7 | `errors` prop with `file: 'content'` entry → decoration applied; `file: 'template'` entry → not applied | component | done |

### StatusBar.test.tsx

| # | Description | Type | Status |
|---|---|---|---|
| RT8 | No errors → shows green/ok validation indicator | component | done |
| RT9 | One error → shows first error message text | component | done |
| RT10 | Shows page count from props | component | done |
| RT11 | Shows shaping engine string from props | component | done |

### CompileButton.test.tsx

| # | Description | Type | Status |
|---|---|---|---|
| RT12 | Button enabled when not compiling and no errors | component | done |
| RT13 | Button disabled when `isCompiling` is true | component | done |
| RT14 | Click calls `window.studio.triggerPdfCompile()` exactly once | component | done |

### error-decorations.test.ts

| # | Description | Type | Status |
|---|---|---|---|
| RT15 | Empty errors array → no decorations in CM6 StateField | unit | done |
| RT16 | One error at line 3 → Decoration.mark spans the correct character range on line 3 | unit | done |
| RT17 | Error at line beyond document length → decoration is silently clamped / not added (no throw) | unit | done |

## Acceptance

- TemplatePanel and ContentPanel render CodeMirror 6 editors with correct language
  syntax highlighting. Editors are read-only. Error underlines appear on the reported lines.
- StatusBar reflects the current compile state. StaleBanner appears over the last valid
  preview when errors are present.
- "Compile!" button triggers a full PDF compile, the PDF is written to `output/`, and
  the system PDF viewer opens it.
- All five PoC acceptance criteria from the brainstorm are demonstrable end-to-end.
- All 17 required tests pass. No existing monorepo tests regress.

## Changelog

- 260508-1030 — plan created
