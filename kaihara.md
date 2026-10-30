# Arahan Serahan Kerja untuk Antigravity — EZOffice Front-End

> **Cara guna:** Hantar **Mesej 0** dahulu (sekali sahaja, di awal). Selepas itu, hantar
> setiap "Brief" satu-satu ikut susunan, tunggu Antigravity siapkan dan report, semak
> dengan checklist, baru hantar brief seterusnya. **Jangan hantar dua brief serentak.**

---

## MESEJ 0 — Hantar Dahulu (Sekali Sahaja, Sebelum Task Pertama)

```
Saya ada projek EZOffice (Electron + React 19 + TypeScript strict + Vite 8 + Tailwind v4).
Saya akan hantar task front-end kepada awak SATU PERSATU, ikut fasa. Jangan mula kerja
sebelum saya hantar brief task yang spesifik.

SEBELUM APA-APA KERJA, baca dan faham SEMPADAN ni — ia mutlak, tiada pengecualian:

DILARANG SAMA SEKALI DISENTUH:
- electron/** (semua fail — main process, IPC handlers, services, DB, migrations)
- src/shared/types/** (kontrak API preload <-> renderer, Zod schema)
- Validation logic (Zod schema di src/shared/types/inputs.ts)
- Calculation logic (payroll engine, attendance aggregation, statutory rates)
- Argumen yang dihantar ke window.api.* (mutation calls)
- Susunan state reset selepas operasi (business-meaningful)
- electron-builder.yml, vite.config.ts, package.json (deps/build)
- CLAUDE.md, ARCHITECTURE.md

BOLEH DIUBAH (domain awak):
- src/shared/components/** — cipta/ubah komponen
- src/index.css — token @theme, dark mode
- src/App.tsx — WRAPPER/ROUTING/LAYOUT sahaja (ErrorBoundary, dark class, providers).
  Jangan sentuh auth state logic, IPC calls, business handlers.
- src/shared/hooks/** — hook UI-only sahaja. Jangan ubah useIpcQuery.ts CONTRACT
  (boleh tambah presentation logic macam toast trigger).
- src/modules/**/*.tsx — JSX, styling, className, presentation logic (modal/tab state,
  confirm trigger, toast trigger)
- src/modules/**/constants.ts — peta tone/label display

KAWASAN KELABU (WAJIB TANYA SAYA DULU):
- Nak tambah npm dependency baru → tanya dulu
- Nak ubah preload.ts API surface → jangan, minta saya tambah channel IPC baru
- Nak ubah skema warna utama (Indigo) → dilarang, locked

PERATURAN AM:
- Satu perubahan logik = satu commit, guna Conventional Commits (feat:, fix:, refactor:,
  chore:, style:)
- Sebelum report siap, WAJIB run:
    npx tsc -p tsconfig.app.json --noEmit
    npx tsc -p tsconfig.node.json --noEmit
  Kedua-dua mesti 0 error. Tiada `any` baru.
- Bila report balik, sertakan: senarai fail yang diubah + hasil typecheck + sebarang
  keputusan yang awak buat yang saya patut tahu.

Faham? Saya akan hantar task pertama (FE-1.1) selepas ini.
```

---

## FASA FE-1 — Foundation & Cleanup

### Brief FE-1.1 — Satukan Sistem Toast

```
TASK: FE-1.1 — Satukan Toast (buang duplikasi)

MASALAH:
Ada 2 fail Toast bercanggah:
- src/shared/components/Toast/Toast.tsx — AKTIF sekarang, tapi guna kelas
  "animate-in fade-in slide-in-from-right" dari plugin tailwindcss-animate yang TAK
  dipasang → animasi broken.
- src/shared/components/Toast/ToastProvider.tsx — dead code (tak dipanggil), tapi
  struktur & styling lebih baik (rounded-xl, border-l-4, pointer-events-none container).

Toast/index.ts export dari ./Toast (versi lama/broken).

TUGAS:
1. Guna ToastProvider.tsx sebagai asas.
2. Buang Toast.tsx (versi lama).
3. Namakan semula ToastProvider.tsx → Toast.tsx.
4. Kemas kini Toast/index.ts export dari ./Toast (fail baru).
5. Tambah type "warning" (versi baru sekarang cuma ada success/error/info).
6. JANGAN UBAH signature useToast() — mesti kekal: addToast(message: string, tone?: ToastType).
   6 fail lain import useToast dari '@/shared/components/Toast', jangan pecahkan mereka.

FAIL TERLIBAT:
- src/shared/components/Toast/Toast.tsx (ganti kandungan)
- src/shared/components/Toast/ToastProvider.tsx (buang)
- src/shared/components/Toast/index.ts (kemas kini)

KRITERIA PENERIMAAN:
[ ] Satu fail Toast sahaja kekal
[ ] useToast() API tidak berubah
[ ] success/error/info/warning semua berfungsi
[ ] Toast di top-right, auto-dismiss 4.5s, boleh dismiss manual
[ ] Container pointer-events-none, toast individu pointer-events-auto
[ ] Typecheck 0 error
[ ] App boot okay, toast muncul bila SettingsPage save

AMARAN:
- Ini kerja presentation SAHAJA. Jangan ubah bila/macam mana toast dipanggil dari page lain
  buat masa ini (itu FE-3.1).
- Jangan sentuh electron/**, src/shared/types/**.
```

### Brief FE-1.2 — Global Error Boundary

```
TASK: FE-1.2 — Tambah Global Error Boundary

MASALAH:
Tiada React Error Boundary di root. Komponen throw → white-screen tanpa maklumat.

TUGAS:
1. Cipta src/shared/components/ErrorBoundary/ErrorBoundary.tsx — class component
   (mesti class, React limitation). Fallback UI:
   - Icon error (SVG inline sahaja — JANGAN tambah dependency icon library)
   - Mesej "Something went wrong"
   - Butang "Reload" → window.location.reload()
   - Butang "Copy error" → copy error.message + componentStack ke clipboard
2. Eksport dari ErrorBoundary/index.ts dan src/shared/components/index.ts.
3. Bungkus root tree di src/App.tsx dengan <ErrorBoundary> — DI LUAR semua provider
   (QueryClientProvider, HashRouter). Paling luar sekali.
4. Fallback guna design system tokens (bg-background, text-neutral-900, Button component)
   — jangan hex mentah.

FAIL TERLIBAT:
- src/shared/components/ErrorBoundary/ErrorBoundary.tsx (baru)
- src/shared/components/ErrorBoundary/index.ts (baru)
- src/shared/components/index.ts (tambah export)
- src/App.tsx (TAMBAH WRAPPER SAHAJA)

KRITERIA PENERIMAAN:
[ ] Throw di mana-mana komponen → fallback muncul, bukan blank screen
[ ] Butang Reload berfungsi
[ ] Butang Copy error menyalin ke clipboard
[ ] Fallback guna design tokens
[ ] Typecheck 0 error

AMARAN — App.tsx SANGAT SENSITIF:
- Anda HANYA boleh tambah wrapper <ErrorBoundary> di App.tsx.
- JANGAN ubah: auth state logic, handleLoginSuccess, handleLogout, checkFirstLaunch effect,
  IPC calls, business handlers — walaupun sebaris.
- Kalau ragu sama ada satu baris di App.tsx itu "layout" atau "business logic", TANYA SAYA
  dulu sebelum ubah — jangan andaikan.
```

### Brief FE-1.3 — Betulkan Inconsistent Radius

```
TASK: FE-1.3 — Betulkan radius tak konsisten

MASALAH:
Design system: radius-sm=8px, radius-md=12px, radius-lg=16px, radius-xl=20px.
2 lokasi salah:
- src/modules/auth/LoginPage.tsx:97 (card) — sekarang rounded-lg, patut rounded-xl
- src/shared/components/SettingsPage.tsx:173 (raw input) — sekarang rounded-lg, patut
  rounded-md (match Input component)

TUGAS: Tukar HANYA 2 lokasi di atas.

FAIL TERLIBAT: LoginPage.tsx, SettingsPage.tsx

KRITERIA PENERIMAAN:
[ ] Login card guna rounded-xl
[ ] SettingsPage raw input guna rounded-md
[ ] Tiada perubahan visual pada komponen lain

AMARAN:
- JANGAN ubah rounded-sm (8px) pada error message — itu sengaja, sudah betul.
- Ini task kecil — jangan "sambil lalu" refactor benda lain dalam fail yang sama.
```

---

## FASA FE-2 — Shared Component Hardening

> Nota untuk anda (Kaihara): FE-2.1 dan FE-2.2 adalah prasyarat untuk FE-3. Pastikan FE-2.1
> siap dan lulus sebelum hantar FE-3.1/FE-3.2.

### Brief FE-2.1 — Komponen ConfirmDialog

```
TASK: FE-2.1 — Bina komponen ConfirmDialog (reusable)

MASALAH:
8 lokasi guna confirm() native — nampak janggal, tak konsisten dengan design system.
(Senarai penuh 8 lokasi akan saya bagi dalam task FE-3.2 — buat masa ini fokus bina
komponen sahaja.)

TUGAS:
1. Cipta src/shared/components/ConfirmDialog/ConfirmDialog.tsx — bina ATAS Modal sedia ada.
   Props:
   interface ConfirmDialogProps {
     isOpen: boolean
     title: string
     message: ReactNode
     confirmLabel?: string      // default "Confirm"
     cancelLabel?: string       // default "Cancel"
     tone?: 'danger' | 'primary'
     onConfirm: () => void
     onCancel: () => void
   }
2. Guna Modal dengan size="sm", footer ada Cancel (secondary) + Confirm (danger/primary
   ikut tone).
3. Eksport dari index.ts masing-masing dan src/shared/components/index.ts.
4. JANGAN wire ke mana-mana page lagi — itu task lain (FE-3.2), akan saya hantar kemudian.

FAIL TERLIBAT:
- src/shared/components/ConfirmDialog/ConfirmDialog.tsx (baru)
- src/shared/components/ConfirmDialog/index.ts (baru)
- src/shared/components/index.ts (tambah export)

KRITERIA PENERIMAAN:
[ ] Render dengan Modal sedia ada (Esc + backdrop close berfungsi)
[ ] tone="danger" → butang confirm variant="danger"
[ ] tone="primary" → butang confirm variant="primary"
[ ] Label boleh di-override
[ ] Typecheck 0 error

AMARAN:
- Komponen ini BERDIRI SENDIRI buat masa ini — jangan sentuh mana-mana page CRUD lagi.
- Jangan tambah dependency baru tanpa tanya saya dulu.
```

### Brief FE-2.2 — Modal: Focus Trap + Focus Restoration

```
TASK: FE-2.2 — Modal focus trap & focus restoration

MASALAH:
src/shared/components/Modal/Modal.tsx ada aria-modal, aria-labelledby, Esc close, backdrop
close — tapi:
- Tiada focus trap (Tab boleh lompat keluar modal ke elemen di belakang)
- Tiada focus restoration (bila modal tutup, focus tak kembali ke elemen pencetus)
- Tiada auto-focus ke dalam modal bila buka

TUGAS:
1. Bila modal buka: simpan document.activeElement (elemen pencetus), fokus ke modal
   container atau elemen boleh-fokus pertama di dalamnya.
2. Trap Tab/Shift+Tab dalam modal — Tab di hujung pusing ke mula, Shift+Tab di mula pusing
   ke hujung.
3. Bila modal tutup: restore fokus ke elemen pencetus yang disimpan.
4. Implementasi manual dibenarkan (querySelector:
   button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])).
   JANGAN tambah dependency (cth. focus-trap-react) tanpa tanya saya dulu.
5. Boleh ekstrak ke hook src/shared/hooks/useFocusTrap.ts untuk reusability.

FAIL TERLIBAT:
- src/shared/components/Modal/Modal.tsx
- src/shared/hooks/useFocusTrap.ts (baru, jika ekstrak)

KRITERIA PENERIMAAN:
[ ] Buka modal → fokus auto masuk dalam modal
[ ] Tab/Shift+Tab kekal dalam modal
[ ] Tutup modal (Esc/backdrop/button) → fokus kembali ke elemen pencetus
[ ] Test dengan EmployeeForm, AttendanceLogForm — berfungsi
[ ] Typecheck 0 error

AMARAN:
- Modal.tsx dipakai di banyak tempat — test SEMUA modal yang guna komponen ni sebelum
  report siap, bukan satu sahaja.
- Jangan tambah npm dependency tanpa kelulusan saya.
```

### Brief FE-2.3 — Table: aria-sort

```
TASK: FE-2.3 — Tambah aria-sort pada Table

MASALAH:
src/shared/components/Table/Table.tsx ada sortable columns tapi <th> tiada atribut
aria-sort — screen reader tak tahu column mana sedang di-sort.

TUGAS:
1. Pada setiap <th> sortable, tambah aria-sort:
   - 'none' jika bukan column aktif
   - 'ascending' jika sortKey === column.key && sortDirection === 'asc'
   - 'descending' jika sortKey === column.key && sortDirection === 'desc'
2. Column tak sortable — aria-sort tak perlu / omit.
3. Semak button sort sedia ada ada role="button" dan tabIndex={0} — kalau belum, tambah.

FAIL TERLIBAT: src/shared/components/Table/Table.tsx

KRITERIA PENERIMAAN:
[ ] Inspect <th> sortable → ada aria-sort dengan nilai betul
[ ] Klik sort → aria-sort berubah (asc/desc/none)
[ ] Typecheck 0 error

AMARAN:
- Ini perubahan attribute sahaja — jangan ubah logic sorting yang sedia ada.
```

### Brief FE-2.4 — Ganti Komponen Mentah dengan Shared Components

```
TASK: FE-2.4 — Ganti raw markup dengan shared components

MASALAH:
1. src/modules/audit/AuditLogPage.tsx:42 — badge manual dengan template literal colors,
   patut guna StatusBadge.
2. src/shared/components/SettingsPage.tsx:173 — raw <input> styling hardcoded, bypass
   Input component. File upload juga raw.

TUGAS:
1. AuditLogPage: tentukan mapping action → BadgeTone (cth. create/update/delete →
   info/warning/error, login/logout → neutral). Ganti badge manual dengan
   <StatusBadge tone={...}>{label}</StatusBadge>. Boleh tambah
   src/modules/audit/constants.ts (baru) ikut pola src/modules/attendance/constants.ts.
2. SettingsPage: ganti raw <input> text/email/phone dengan <Input> component. Untuk file
   upload (logo), bina src/shared/components/Input/FileInput.tsx baru (atau tambah variant
   ke Input — pilih yang lebih bersih). Guna `file:` modifier Tailwind sedia ada, bungkus
   dalam Field.

FAIL TERLIBAT:
- src/modules/audit/AuditLogPage.tsx
- src/modules/audit/constants.ts (baru)
- src/shared/components/SettingsPage.tsx
- src/shared/components/Input/FileInput.tsx (baru, jika berasingan)
- src/shared/components/Input/index.ts (tambah export jika perlu)

KRITERIA PENERIMAAN:
[ ] AuditLogPage badge guna StatusBadge, tiada template literal color
[ ] SettingsPage text inputs guna Input component
[ ] File upload berfungsi (upload, preview, validate 2MB)
[ ] Tiada regression visual
[ ] Typecheck 0 error

AMARAN:
- SettingsPage ada logic save/validation — anda HANYA ubah markup/component swap, JANGAN
  ubah validation logic atau apa yang dihantar semasa save.
- Validate 2MB untuk file upload — kalau logic validate ni sedia ada di business layer,
  jangan tulis semula, panggil sahaja.
```

---

## FASA FE-3 — UX Feedback & States

> Bergantung pada: FE-1.1 (Toast) + FE-2.1 (ConfirmDialog) mesti siap dan lulus dahulu.
> INI FASA PALING SENSITIF — sentuh CRUD pages dan mutation triggers. Semak diff dengan
> teliti selepas setiap task.

### Brief FE-3.1 — Wire Success Toast pada Semua CRUD

```
TASK: FE-3.1 — Wire success toast pada semua operasi CRUD

MASALAH:
useIpcMutation (src/shared/hooks/useIpcQuery.ts) ada option onSuccessMessage tapi
majoriti page tak guna. Create/update/delete employee/customer/supplier/product — tiada
toast success. User tiada feedback.

TUGAS:
1. Pada setiap useIpcMutation call di page CRUD, tambah option:
   { onSuccessMessage: 'Employee created successfully' }  // sesuaikan mesej
2. PENTING: useIpcMutation perlu diubah untuk MEMBACA onSuccessMessage dan trigger toast.
   Semak implementasi semasa — kalau ia terima option tapi tak guna, tambah logic panggil
   useToast().addToast(message, 'success') dalam onSuccess. Ini presentation concern
   sahaja (dibenarkan), BUKAN business logic.
3. Mesej spesifik: "Employee created", "Customer updated", "Product deleted" — bukan
   generik "Success".

FAIL TERLIBAT:
- src/modules/master-data/employees/EmployeeListPage.tsx
- src/modules/master-data/customers/CustomerListPage.tsx
- src/modules/master-data/suppliers/SupplierListPage.tsx
- src/modules/master-data/products/ProductListPage.tsx
- src/modules/attendance/AttendanceListPage.tsx (+ ShiftManagementPanel, LeaveApprovalPanel)
- src/modules/payroll/salaryStructures/SalaryStructureListPage.tsx
- src/modules/payroll/salaryAdvances/SalaryAdvanceListPage.tsx
- src/modules/payroll/rateTables/* (jika ada mutation)
- src/shared/hooks/useIpcQuery.ts (tambah toast trigger — PRESENTATION SAHAJA)

KRITERIA PENERIMAAN:
[ ] Setiap create/update/delete berjaya → toast success muncul
[ ] Mesej spesifik per entiti
[ ] Toast hilang auto selepas 4.5s
[ ] BUSINESS LOGIC MUTATION TAK BERUBAH (argumen, endpoint, state reset kekal sama)

AMARAN — INI PALING KRITIKAL:
- Di useIpcQuery.ts, anda HANYA boleh tambah logic "panggil toast bila berjaya" dalam
  onSuccess. JANGAN ubah: apa yang dihantar ke window.api.*, mutation function itu sendiri,
  query key, cache invalidation logic, atau error handling sedia ada.
- Di setiap page, JANGAN ubah argumen yang dihantar ke mutateAsync(), JANGAN ubah susunan
  setState selepas mutation berjaya — hanya TAMBAH option onSuccessMessage.
- Kalau useIpcMutation punya struktur buat ni jadi rumit atau perlukan ubah signature yang
  boleh jejaskan fail lain, STOP dan tanya saya dulu — jangan teruskan dengan assumption.
- Selepas siap, sertakan diff untuk useIpcQuery.ts secara berasingan dalam report supaya
  saya boleh semak teliti.
```

### Brief FE-3.2 — Ganti confirm() dengan ConfirmDialog

```
TASK: FE-3.2 — Ganti semua confirm() native dengan ConfirmDialog

MASALAH:
8 lokasi guna confirm() native:
| Fail | Baris | Mesej |
|------|-------|-------|
| EmployeeListPage.tsx | 96 | Delete employee "${name}"? |
| CustomerListPage.tsx | 57 | Delete customer "${name}"? |
| SupplierListPage.tsx | 71 | Delete supplier "${name}"? |
| ProductListPage.tsx | 64 | Delete product "${name}"? |
| AttendanceListPage.tsx | 222 | Delete this attendance log? |
| ShiftManagementPanel.tsx | 72 | Delete shift "${name}"? |
| SalaryAdvanceListPage.tsx | 91 | Delete this salary advance? |
| SalaryStructureListPage.tsx | 95 | Delete this salary structure? |

TUGAS:
1. Tambah state confirmDialog di setiap page: { isOpen, message, onConfirm }.
2. Ganti "if (!confirm(...)) return" dengan buka dialog. Confirm → panggil handler sebenar
   (mutation). Cancel → tutup dialog, JANGAN panggil mutation.
3. YANG BERUBAH HANYALAH MEKANISME PENGESAHAN (native → custom dialog). YANG TIDAK
   BERUBAH: argumen mutation, mutation yang dipanggil, susunan setIsFormOpen/setEditingX
   selepas delete.

POLA WAJIB IKUT (contoh untuk Employee, ikut pola sama untuk 7 fail lain):

  // SEBELUM
  const handleDelete = useCallback(async () => {
    if (!editingEmployee) return
    if (!confirm(`Delete employee "${editingEmployee.name}"?`)) return   // <- GANTI INI
    await deleteMutation.mutateAsync(editingEmployee.id)                 // <- KEKAL SAMA
    setIsFormOpen(false)                                                 // <- KEKAL SAMA
    setEditingEmployee(null)                                             // <- KEKAL SAMA
  }, [editingEmployee, deleteMutation])

  // SELEPAS
  const [confirmState, setConfirmState] = useState<{
    open: boolean
    message: string
    onConfirm: () => void
  } | null>(null)

  const handleDelete = useCallback(() => {
    if (!editingEmployee) return
    setConfirmState({
      open: true,
      message: `Delete employee "${editingEmployee.name}"? This cannot be undone.`,
      onConfirm: async () => {
        await deleteMutation.mutateAsync(editingEmployee.id)   // <- KEKAL SAMA
        setIsFormOpen(false)                                   // <- KEKAL SAMA
        setEditingEmployee(null)                               // <- KEKAL SAMA
        setConfirmState(null)
      },
    })
  }, [editingEmployee, deleteMutation])

  // Di JSX:
  <ConfirmDialog
    isOpen={confirmState?.open ?? false}
    title="Confirm Delete"
    message={confirmState?.message ?? ''}
    tone="danger"
    confirmLabel="Delete"
    onConfirm={confirmState?.onConfirm ?? (() => {})}
    onCancel={() => setConfirmState(null)}
  />

FAIL TERLIBAT: 8 fail dalam jadual di atas + ConfirmDialog dari FE-2.1.

KRITERIA PENERIMAAN:
[ ] grep "confirm(" dalam src/ — tiada tinggal
[ ] Dialog muncul dengan mesej betul, tone danger
[ ] Confirm → delete berlaku (mutation dipanggil dengan argumen sama macam sebelum)
[ ] Cancel → tiada delete, dialog tutup
[ ] MUTATION CALLS, ARGUMEN, STATE RESET SELEPAS DELETE — KEKAL EXACT SAMA

AMARAN — DIFF REVIEW AKAN FOKUS 100% PADA INI:
- Untuk SETIAP satu daripada 8 fail, buat SATU commit berasingan — supaya saya boleh
  review satu-satu, bukan satu commit besar untuk semua 8.
- JANGAN "improve" atau refactor apa-apa lain dalam fail yang sama semasa buat ni — fokus
  hanya pada confirm() → ConfirmDialog.
- Kalau ada logic tambahan dalam block confirm() yang sedia ada (cth. ada check lain
  selain confirm), JANGAN buang — kekalkan, hanya ganti bahagian confirm() sahaja.
- Report balik dengan diff 8 fail ni SECARA BERASINGAN, bukan digabung.
```

### Brief FE-3.3 — Ganti alert() dengan Toast

```
TASK: FE-3.3 — Ganti alert() dengan toast error

MASALAH:
2 lokasi guna alert():
- EmployeeListPage.tsx:89 — alert(`Export failed: ${err}`)
- AttendanceListPage.tsx:234 — alert(`Export failed: ${err}`)

TUGAS:
Ganti dengan addToast(`Export failed: ${String(err)}`, 'error'). Pastikan page import
useToast.

FAIL TERLIBAT: 2 fail di atas.

KRITERIA PENERIMAAN:
[ ] grep "alert(" dalam src/ — tiada tinggal
[ ] Export gagal → toast error muncul

AMARAN:
- Ganti sahaja mekanisme paparan error — JANGAN ubah logic export/catch block itu sendiri.
```

### Brief FE-3.4 — Enhanced Empty States

```
TASK: FE-3.4 — Empty state dengan illustration/icon

MASALAH:
Table empty state hanya text — nampak kosong dan kurang polished.

TUGAS:
1. Tambah prop emptyState.icon? (ReactNode) ke Table. SVG inline ringkas sahaja
   (JANGAN tambah icon library) — cipta 2-3 SVG reusable di
   src/shared/components/icons/ (cth. EmptyBoxIcon, NoResultsIcon).
2. Layout: icon (neutral-300, size-12) di atas, title (text-neutral-700 font-medium),
   description (text-neutral-500), action (jika ada).
3. Setiap page CRUD pass icon sesuai + action butang "Add X" bila senarai kosong.

FAIL TERLIBAT:
- src/shared/components/Table/Table.tsx
- src/shared/components/icons/ (baru)
- Semua page CRUD (pass icon + action ke emptyState)

KRITERIA PENERIMAAN:
[ ] Table kosong → icon + text + (jika ada) action butang
[ ] Icon neutral, tak menarik perhatian berlebihan
[ ] Typecheck 0 error

AMARAN:
- Jangan tambah icon library (lucide, heroicons, dll) tanpa tanya saya — SVG inline sahaja.
- Butang "Add X" pada empty state mesti panggil handler yang SAMA dengan butang "Add" biasa
  di page tu — jangan cipta logic baharu berasingan.
```

### Brief FE-3.5 — Konsistensi Loading States

```
TASK: FE-3.5 — Skeleton loading state

MASALAH:
SettingsPage guna loading text "Loading settings..." berasingan. Page lain guna Table
isLoading skeleton. Tak konsisten.

TUGAS:
1. Cipta src/shared/components/Skeleton/Skeleton.tsx — placeholder berpulse (varian:
   text, card, line). Guna animate-pulse + bg-neutral-200.
2. SettingsPage: ganti "Loading settings..." dengan skeleton layout (form fields skeleton).
3. AuditLogPage: semak ada loading state atau tidak, tambah jika tiada.
4. JANGAN ubah Table skeleton sedia ada (sudah betul) — hanya tambah untuk page yang tiada.

FAIL TERLIBAT:
- src/shared/components/Skeleton/Skeleton.tsx (baru)
- src/shared/components/SettingsPage.tsx
- src/modules/audit/AuditLogPage.tsx (semak)

KRITERIA PENERIMAAN:
[ ] SettingsPage loading → skeleton form, bukan text
[ ] AuditLogPage ada loading state
[ ] Skeleton guna animate-pulse, warna neutral-200

AMARAN:
- Jangan sentuh Table.tsx punya skeleton sedia ada.
```

---

## FASA FE-4 — Dark Mode

> Fasa paling besar effort. Design direction (Indigo/Ink) locked — dark mode ADALAH
> TAMBAHAN, bukan ganti arah design.

### Brief FE-4.1 — Definisi Dark Mode Tokens

```
TASK: FE-4.1 — Dark mode tokens di index.css

MASALAH:
src/index.css ada @theme block token light sahaja. Tiada dark: variant atau override
.dark. Tailwind v4 CSS-first perlukan definisi eksplisit.

TUGAS:
1. Tambah block override di src/index.css (DI LUAR @theme, guna selector .dark):
   .dark {
     --color-background: #0f0f12;
     --color-surface: #18181b;
     --color-border: #27272a;
     --color-neutral-200: #27272a;
     --color-neutral-300: #3f3f46;
     --color-neutral-400: #71717a;
     --color-neutral-500: #a1a1aa;
     --color-neutral-700: #d4d4d8;
     --color-neutral-900: #f4f4f5;
   }
   PENTING: ini override CSS custom properties sahaja. Komponen guna utility class yang
   rujuk token (bg-surface, text-neutral-900) — override akan cascade automatik. JANGAN
   tambah class `dark:` pada setiap komponen satu-satu — itu approach lama, langgar
   prinsip token.
2. Pastikan body background/color rujuk var(--color-background) / var(--color-neutral-900)
   (semak sudah ada).
3. Tambah color-scheme: dark pada .dark untuk native form controls.

FAIL TERLIBAT: src/index.css

KRITERIA PENERIMAAN:
[ ] Toggle dark mode → background, card, border, text berubah
[ ] Light mode kekal tidak berubah
[ ] color-scheme betul (scrollbar gelap di dark mode)

AMARAN:
- JANGAN tukar skema warna utama (Indigo) — cuma tambah dark variant token, bukan ganti
  arah design.
- Pendekatan MESTI token-based (CSS variable override), BUKAN tambah class dark: pada
  setiap komponen — kalau saya nampak dark: bertaburan dalam diff, saya akan minta ubah.
```

### Brief FE-4.2 — Wrap LoginPage dalam Dark Class

```
TASK: FE-4.2 — Betulkan wrapper dark class di App.tsx

MASALAH:
src/App.tsx return LoginPage SEBELUM <div className={isDarkMode ? 'dark' : ''}> — jadi
walaupun CSS wujud (FE-4.1), login screen tak akan dark.

TUGAS:
1. Pindahkan wrapper <div className={isDarkMode ? 'dark' : ''}> ke PALING LUAR — bungkus
   KEDUA-DUA cabang (LoginPage + authenticated) di dalamnya.
2. Struktur betul:
   return (
     <ErrorBoundary>
       <div className={isDarkMode ? 'dark' : ''}>
         <QueryClientProvider client={queryClient}>
           {!auth.isAuthenticated ? <LoginPage ... /> : <HashRouter>...</HashRouter>}
         </QueryClientProvider>
       </div>
     </ErrorBoundary>
   )
3. JANGAN ubah auth state logic, handleLoginSuccess, handleLogout, checkFirstLaunch
   effect — itu business logic.

FAIL TERLIBAT: src/App.tsx

KRITERIA PENERIMAAN:
[ ] Dark mode ON → LoginPage juga dark
[ ] Auth flow (login, signup, first-launch detection) kekal berfungsi
[ ] Initializing screen juga dark bila dark mode ON

AMARAN — App.tsx SEKALI LAGI SANGAT SENSITIF:
- Ini HANYA pindah lokasi satu <div> wrapper. JANGAN ubah apa-apa logic di dalam cabang
  LoginPage atau HashRouter semasa buat ni.
- Test auth flow PENUH (login, logout, first-launch) selepas ubah — bukan visual sahaja —
  sebab App.tsx pegang auth state.
```

### Brief FE-4.3 — Audit & Fix Hardcoded Light Colors

```
TASK: FE-4.3 — Audit hardcoded light colors

MASALAH:
Token override (FE-4.1) cascade ke majoriti komponen, tapi ada hardcoded light colors yang
tak akan berubah:
| Pattern | Lokasi | Fix |
|---------|--------|-----|
| bg-white | Button secondary, Card, Table, Modal, Input, SettingsPage | Ganti bg-surface |
| bg-neutral-100 | App.tsx:104 (initializing) | Ganti bg-background |
| bg-neutral-50 | Table.tsx:140 (hover) | OK / ganti bg-white/5 |
| Sidebar bg-ink-900 | AppShell.tsx | OK, kekal sama |

TUGAS:
1. grep "bg-white" dalam src/ — ganti dengan bg-surface di card, modal, input, table,
   button secondary. JANGAN ganti tempat yang memang patut putih (cth. logo circle).
2. grep "bg-neutral-100" di App.tsx initializing screen → bg-background.
3. Test setiap screen di dark mode, catat mana masih light/broken, fix satu-satu.

FAIL TERLIBAT: Semua fail src/shared/components/* + page dengan bg-white hardcoded.

KRITERIA PENERIMAAN:
[ ] grep "bg-white" src/ → tinggal hanya yang sengaja putih (logo, dll)
[ ] Semua 6 tab Attendance, 5 tab Payroll, 4 Master Data, Audit, Settings — render betul
    di dark mode
[ ] Light mode kekal tidak berubah (regression test)
[ ] Tiada class dark: ditambah (approach token, bukan per-komponen)

AMARAN:
- Ini audit visual sahaja — jangan ubah struktur JSX atau logic komponen semasa buat ni.
- Sertakan senarai fail yang diubah dalam report — banyak fail terlibat, saya perlu tahu
  exact list untuk spot check.
```

### Brief FE-4.4 — Focus Ring Offset untuk Dark Mode

```
TASK: FE-4.4 — Focus ring offset dark mode

MASALAH:
Button guna focus-visible:ring-offset-2 tanpa ring-offset-color — default white. Di dark
mode, offset ni tak kelihatan pada background gelap.

TUGAS:
1. Tambah focus-visible:ring-offset-background (atau ring-offset-surface) di Button.tsx.
2. Jika token --color-background/--color-surface tak auto-jana utility ring-offset-*,
   tambah class custom atau guna ring-offset-2 ring-offset-[var(--color-surface)].
3. Semak Input, Select — ada focus ring juga, pastikan offset betul.

FAIL TERLIBAT: Button.tsx, Input.tsx, Select.tsx

KRITERIA PENERIMAAN:
[ ] Focus ring visible di light AND dark mode
[ ] Offset warna match background

AMARAN:
- Perubahan kecil, styling sahaja — jangan sentuh event handler atau logic komponen.
```

---

## FASA FE-5 — Motion & Polish

> Boleh jalan selari dengan FE-3/FE-4 kalau nak, tiada kebergantungan ketat.

### Brief FE-5.1 — Toast Enter/Exit Animations

```
TASK: FE-5.1 — Animasi toast masuk/keluar

TUGAS: Tambah keyframes di src/index.css:
  @keyframes toast-in {
    from { opacity: 0; transform: translateX(100%); }
    to   { opacity: 1; transform: translateX(0); }
  }
  @keyframes toast-out {
    from { opacity: 1; transform: translateX(0); }
    to   { opacity: 0; transform: translateX(100%); }
  }
Guna pada toast container: animate-[toast-in_0.2s_ease-out]. Untuk exit, perlu state
"leaving" — tambah flag di Toast state, delay remove sehingga animasi tamat (~200ms).

FAIL TERLIBAT: src/index.css, src/shared/components/Toast/Toast.tsx

AMARAN:
- Jangan ubah useToast() API atau logic auto-dismiss timing (4.5s) yang dah siap dari
  FE-1.1 — ini tambah animasi visual sahaja.
```

### Brief FE-5.2 — Tab Switch Transitions

```
TASK: FE-5.2 — Fade transition bila switch tab

TUGAS: AttendanceListPage (6 tab) dan PayrollListPage (5 tab) — tambah fade transition
ringan bila switch tab. TANPA library (jangan guna framer-motion tanpa tanya saya dulu).
Pendekatan: <div key={activeTab} className="animate-[fade-in_0.15s_ease-out]"> di
sekeliling content tab.

FAIL TERLIBAT: AttendanceListPage.tsx, PayrollListPage.tsx

AMARAN:
- Ini wrapper visual sahaja di sekeliling tab content — JANGAN ubah tab state logic atau
  data fetching per tab.
```

### Brief FE-5.3 — Page (Route) Transitions

```
TASK: FE-5.3 — Fade transition antara route

TUGAS: Tambah fade transition pada <Outlet> di AppShell. Guna useLocation() key sebagai
trigger, wrap outlet dalam div dengan key={location.pathname} + animate fade-in.
TANPA library.

FAIL TERLIBAT: src/shared/components/AppShell.tsx, src/index.css (keyframe fade-in)

AMARAN:
- Jangan ubah routing logic (HashRouter routes) — hanya tambah wrapper visual di sekeliling
  Outlet.
```

### Brief FE-5.4 — Initializing Screen Polish

```
TASK: FE-5.4 — Polish initializing screen

MASALAH:
App.tsx initializing screen hanya text "Initializing..." — tiada spinner/branding.

TUGAS: Ganti dengan layout branded: logo circle "EZ" (sama macam sidebar), spinner (guna
Spinner dari Button atau extract ke shared component), text "EZOffice" + "Loading...".
Guna bg-background (bukan bg-neutral-100) supaya dark-mode ready.

FAIL TERLIBAT: src/App.tsx (initializing branch SAHAJA), mungkin extract Spinner ke
src/shared/components/Spinner/.

AMARAN:
- App.tsx sekali lagi — HANYA ubah dalam initializing branch punya JSX/visual. JANGAN
  sentuh checkFirstLaunch effect atau logic yang tentukan bila initializing screen tunjuk.
```

---

## FASA FE-6 — Advanced UX (Optional, Prioriti Rendah)

> Hanya hantar fasa ni selepas FE-1 hingga FE-5 selesai dan lulus semua.

### Brief FE-6.1 — Collapsible Sidebar

```
TASK: FE-6.1 — Sidebar collapsible

MASALAH: Sidebar fixed 240px, makan ruang pada window kecil.

TUGAS:
1. Tambah state isSidebarCollapsed di AppShell (default false).
2. Collapse → sidebar jadi 64px, icon-only (sembunyi label, tooltip hover).
3. Butang toggle (hamburger/chevron) di header sidebar.
4. Persist state ke localStorage (key sidebarCollapsed).
5. Refaktor nav item untuk support icon-only mode.

Perlu set SVG icon inline untuk setiap nav item (Employees, Customers, Suppliers,
Products, Attendance, Payroll, ERP, Audit, Settings) di src/shared/components/icons/.
JANGAN tambah icon library tanpa kelulusan saya.

FAIL TERLIBAT: AppShell.tsx, src/shared/components/icons/ (baru)

AMARAN:
- Nav routing/link logic KEKAL SAMA — hanya ubah presentation (icon-only vs full label).
- localStorage untuk UI preference sahaja (collapsed state) — bukan untuk data business.
```

### Brief FE-6.2 — Keyboard Shortcuts

```
TASK: FE-6.2 — Keyboard shortcuts global

TUGAS:
1. Cipta src/shared/hooks/useKeyboardShortcut.ts — daftar shortcut global.
2. Shortcut (semua dengan Ctrl/Cmd):
   - Ctrl+N — trigger "Add" pada page aktif
   - Esc — tutup form/modal aktif
   - Ctrl+S — save form aktif
3. Daftar di AppShell atau per-page.

FAIL TERLIBAT: src/shared/hooks/useKeyboardShortcut.ts (baru), AppShell.tsx, page forms.

AMARAN — WAJIB TANYA SAYA DULU SEBELUM IMPLEMENT:
- JANGAN override shortcut Electron built-in — kalau ragu shortcut ni clash dengan apa-apa
  di electron/**, TANYA SAYA dulu sebelum teruskan. Ini boleh terjejas main process punya
  keyboard handling.
- Ctrl+S "save form aktif" — pastikan ini trigger handler UI sahaja (macam klik butang
  Save sedia ada), BUKAN cipta logic save baharu.
```

### Brief FE-6.3 — Responsive Window Sizing

```
TASK: FE-6.3 — Audit responsive width

TUGAS:
1. Test setiap page pada width 800px, 1024px, 1280px, 1920px.
2. Table horizontal scroll sudah ada (overflow-x-auto) — OK, tak perlu ubah.
3. PageHeader actions mungkin overflow pada width kecil — tambah flex-wrap atau collapse
   ke menu.
4. Form grid (jika multi-column) — pastikan stack pada width kecil.
5. Catat breakpoint yang break, fix satu-satu.

FAIL TERLIBAT: Audit semua page, fix yang break.

AMARAN:
- Ini styling/layout audit sahaja — jangan ubah form field logic, validation, atau apa-apa
  yang bukan pure CSS/layout punca masalah.
- Report balik senarai page + breakpoint yang bermasalah SEBELUM fix, kalau list panjang —
  saya nak prioritize mana yang penting dulu.
```

---

## Checklist Ringkas Semasa Semak Report Antigravity (Setiap Fasa)

Sebelum terima report sebagai "selesai" dan hantar brief seterusnya, semak:

- [ ] `npx tsc -p tsconfig.app.json --noEmit` → 0 error
- [ ] `npx tsc -p tsconfig.node.json --noEmit` → 0 error
- [ ] `git diff --stat electron/` → KOSONG (tiada perubahan)
- [ ] `git diff --stat src/shared/types/` → KOSONG (tiada perubahan)
- [ ] App boot okay (`npm run electron:dev`)
- [ ] Untuk FE-3.1/FE-3.2: semak mutation argumen & state reset TAK berubah dalam diff
- [ ] Tiada dependency baru ditambah dalam package.json tanpa awak approve dulu
- [ ] Commit ikut Conventional Commits, satu logic per commit
