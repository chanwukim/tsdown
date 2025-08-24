# tsdown

This repository reproduces a **tsdown** issue where re-exported types from external packages

## 🐛 The Problem

When using `VariantProps` from `class-variance-authority` through a re-exported package, tsdown generates incorrect type declarations:

**Current Output:**

<img width="1101" height="536" alt="image-1" src="https://github.com/user-attachments/assets/87569413-d3f6-421d-96d3-e27536fbce4f" />

```typescript
// buttonVariants types are correct ✅
declare const buttonVariants: (props?: ({
  variant?: "default" | "destructive" | "outline" | "secondary" | "ghost" | "link" | null | undefined;
  size?: "default" | "sm" | "lg" | "icon" | null | undefined;
} & class_variance_authority_types0.ClassProp) | undefined) => string;

// But Button's variant parameter becomes any ❌
declare function Button({
  variant,  // <- This is 'any' instead of the union type
  ...
}: React.ComponentProps<"button"> & VariantProps<typeof buttonVariants> & {
  asChild?: boolean;
}): react_jsx_runtime0.JSX.Element;
```

**Expected:**

- `variant` parameter should be `"default" | "destructive" | "outline" | "secondary" | "ghost" | "link"`
- Proper IntelliSense and type safety in consuming apps

## 🔄 Reproduction Steps

1. **Clone and install**

   ```bash
   git clone https://github.com/chanwukim/tsdown.git
   cd tsdown
   pnpm install
   ```

2. **Build and check types**

   ```bash
   pnpm build
   cat packages/ui-react/button/dist/index.d.ts
   ```

3. **Verify broken IntelliSense**

<img width="717" height="358" alt="image-2" src="https://github.com/user-attachments/assets/874c143e-166c-411d-98bf-b7aea6b899bd" />

   ```bash
   code apps/react/src/App.tsx
   # variant prop shows as 'string' instead of union type
   ```

## 📁 Key Files

- [`packages/ui-theme/src/utils.ts`](./packages/ui-theme/src/utils.ts) - Re-exports `VariantProps` from CVA
- [`packages/ui-react/button/src/button.tsx`](./packages/ui-react/button/src/button.tsx) - Uses `VariantProps<typeof buttonVariants>`
- [`packages/ui-react/button/dist/index.d.ts`](./packages/ui-react/button/dist/index.d.ts) - Generated broken types
- [`apps/react/src/App.tsx`](./apps/react/src/App.tsx) - Consumer with broken IntelliSense

## 🛠️ Environment

- **tsdown**: ^0.14.1
- **class-variance-authority**: ^0.7.1
- **TypeScript**: ^5.9.0
