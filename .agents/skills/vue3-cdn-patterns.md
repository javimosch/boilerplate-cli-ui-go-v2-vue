# Vue 3 CDN Patterns Skill

## Overview

Vue 3 used via CDN (no build step). All components in separate .js files.

## Setup

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.prod.js"></script>
<script src="https://unpkg.com/lucide@latest/dist/umd/lucide.js"></script>
<script src="https://cdn.tailwindcss.com"></script>
```

## Accessing Vue APIs

```javascript
const { createApp, ref, reactive, computed, onMounted, onUpdated, provide, inject } = Vue;
```

## Component Pattern

```javascript
const MyComponent = {
    props: {
        title: String,
        count: Number
    },
    emits: ['update'],
    template: `
        <div class="bg-white p-4 rounded-lg">
            <h2>{{ title }}</h2>
            <p>{{ count }}</p>
            <button @click="$emit('update')">Update</button>
        </div>
    `,
    setup(props, { emit }) {
        // Composition API logic
        return { /* exposed to template */ };
    }
};

// Register
app.component('my-component', MyComponent);
```

## State Management

### Local State

```javascript
setup() {
    const count = ref(0);
    const increment = () => count.value++;
    return { count, increment };
}
```

### Global State (provide/inject)

```javascript
// app.js
const status = ref(null);
provide('status', status);

// Any component
const status = inject('status');
```

## Lifecycle

```javascript
setup() {
    onMounted(() => {
        // DOM ready
        lucide.createIcons();
    });
    
    onUpdated(() => {
        // After re-render
        lucide.createIcons();
    });
}
```

## Lucide Icons

```html
<!-- In template -->
<i data-lucide="settings" class="w-5 h-5"></i>

<!-- Re-render after dynamic content -->
Vue.onMounted(() => lucide.createIcons());
Vue.onUpdated(() => lucide.createIcons());
```

## Common Pitfalls

### 1. Component Load Order

Load dependencies first:
```html
<script src="/js/components/StatusCard.js"></script>  <!-- Before -->
<script src="/js/views/Dashboard.js"></script>         <!-- After -->
```

### 2. Template References

Templates must be strings (not DOM elements):
```javascript
// Correct
template: `<div>{{ msg }}</div>`

// Wrong
template: document.getElementById('my-template')
```

### 3. Reactivity

Use `ref()` for primitives, `reactive()` for objects:
```javascript
const count = ref(0);        // count.value
const user = reactive({});   // user.name
```

### 4. Event Handling

```javascript
// In template
@click="handleClick"
@click="count++"

// In setup
const handleClick = () => { /* ... */ };
return { handleClick };
```

## Adding a View

1. Create `ui/js/views/MyView.js`:
```javascript
const MyView = {
    template: `<div>My View</div>`,
    setup() { return {}; }
};
```

2. Register in `app.js`:
```javascript
app.component('my-view', MyView);
```

3. Use in `AppLayout.js`:
```html
<my-view v-if="currentView === 'my-view'"></my-view>
```

4. Add nav item:
```javascript
{ id: 'my-view', label: 'My View', icon: 'star' }
```

5. Include in `index.html`:
```html
<script src="/js/views/MyView.js"></script>
```
