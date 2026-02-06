# @mhs-fe/css Module Import Issue Analysis

## Problem Summary (问题总结)

The issue occurs when trying to import directly from `@mhs-fe/css/maps` instead of `@mhs-fe/css/maps/colors`. This is related to **Sass module system constraints** (Sass模块系统约束) and **circular dependency conflicts** (循环依赖冲突).

## Root Cause Analysis (根本原因分析)

### 1. Module Configuration Conflict (模块配置冲突)

**Key Issue**: `@mhs-fe/css` is loaded with configuration in one place and without it in another:

```scss
// In index.scss - WITH configuration
@use "@mhs-fe/css" with ($root: body, $color-space: hsl);

// In _colors.scss - WITHOUT configuration (attempting to use maps)
@use "@mhs-fe/css/maps" as mhs-map;
```

**Sass Rule**: Once a module is loaded with configuration, it cannot be reconfigured or loaded without configuration elsewhere in the same compilation.

### 2. Internal Circular Dependencies (内部循环依赖)

The `@mhs-fe/css` package has internal circular dependencies:

```
init.scss → maps → topography → vars → init.scss
```

This creates a **dependency loop** (依赖循环) that Sass cannot resolve when modules are loaded in different contexts.

### 3. Module System Design Flaw (模块系统设计缺陷)

The current structure violates Sass module system best practices:
- **Mixed forwarding and usage patterns** (混合转发和使用模式)
- **Configuration dependencies across submodules** (跨子模块的配置依赖)
- **Lack of clear module boundaries** (缺乏清晰的模块边界)

## Why `@mhs-fe/css/maps/colors` Works (为什么直接导入colors有效)

When you import `@mhs-fe/css/maps/colors` directly:

```scss
@use "@mhs-fe/css/maps/colors" as colors;
```

This works because:
1. **Direct module access** (直接模块访问) - bypasses the main index.scss configuration
2. **No circular dependency** (无循环依赖) - colors.scss has minimal dependencies
3. **Independent compilation** (独立编译) - doesn't trigger the main module's configuration conflicts

## Solutions (解决方案)

### Solution 1: Create Configuration Module (创建配置模块)

**Recommended approach** (推荐方法):

```scss
// Create: src/styles/_config.scss
@use "@mhs-fe/css" with (
  $root: body,
  $color-space: hsl
) as mhs-configured;

// Forward the configured module
@forward "@mhs-fe/css" with (
  $root: body,
  $color-space: hsl
);
```

```scss
// In _colors.scss
@use "../config" as config;
// Now you can access maps through the configured module
```

### Solution 2: Use Specific Imports (使用特定导入)

**Current working solution** (当前有效解决方案):

```scss
// Instead of @use "@mhs-fe/css/maps"
@use "@mhs-fe/css/maps/colors" as colors;
@use "@mhs-fe/css/maps/topography" as typography;
// Import only what you need
```

### Solution 3: Package Structure Improvement (包结构改进)

**For @mhs-fe/css maintainers** (给@mhs-fe/css维护者):

1. **Separate configuration from core modules** (分离配置和核心模块)
2. **Remove circular dependencies** (移除循环依赖)
3. **Create clear module boundaries** (创建清晰的模块边界)

```scss
// Improved structure
@mhs-fe/css/
├── core/           # Core modules without configuration
│   ├── maps/
│   ├── mixins/
│   └── functions/
├── configured/     # Pre-configured modules
└── index.scss      # Main entry point
```

## Best Practices (最佳实践)

### For Current Usage (当前使用)

1. **Use specific imports** (使用特定导入) when you only need certain maps
2. **Create a configuration module** (创建配置模块) for consistent setup
3. **Avoid mixing configured and unconfigured imports** (避免混合配置和未配置的导入)

### For Package Design (包设计)

1. **Separate concerns** (关注点分离) - configuration vs. core functionality
2. **Avoid circular dependencies** (避免循环依赖)
3. **Use @forward strategically** (策略性使用@forward)
4. **Document module boundaries** (文档化模块边界)

## Technical Terms (技术术语)

- **Module System** (模块系统): Sass's @use/@forward system for organizing code
- **Configuration Constraint** (配置约束): Sass rule preventing module reconfiguration
- **Circular Dependency** (循环依赖): When modules depend on each other in a loop
- **Dependency Injection** (依赖注入): Passing configuration to modules
- **Module Boundary** (模块边界): Clear separation between different modules

## Conclusion (结论)

The issue stems from **fundamental design problems** (基本设计问题) in the `@mhs-fe/css` package's module system. While the immediate solution is to use specific imports like `@mhs-fe/css/maps/colors`, the **long-term solution** (长期解决方案) requires restructuring the package to follow Sass module system best practices.

**Immediate Action** (立即行动): Use `@mhs-fe/css/maps/colors` for specific imports
**Long-term Action** (长期行动): Consider package restructuring or create a configuration wrapper module