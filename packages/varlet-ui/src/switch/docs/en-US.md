# Switch

### Intro

Switching Selector.

### Install

```js
import { createApp } from 'vue'
import { Switch } from '@varlet/ui'

createApp().use(Switch)
```

### Basic Usage

```html
<var-switch v-model="value" />
```

```javascript
import { ref } from 'vue'

export default {
  setup() {
    const value = ref(true)

    return {
      value
    }
  }
}
```

### Not Available

```html
<var-switch v-model="value" disabled />
<var-switch v-model="value" readonly />
```

### Custom color

```html
<var-switch v-model="value" :ripple="false" />
<var-switch v-model="value" color="#ff9f00" close-color="#f5cb90" />
```

### Size

Use `size` prop to change size of Switch.

```html
<var-switch v-model="value" size="15" />
<var-switch v-model="value" />
<var-switch v-model="value" size="25" />
```

### Loading

```html
<var-switch :model-value="true" loading />
<var-switch :model-value="true" size="25" loading loading-color="#ff9f00" />
```

### Validate value

Verify the value by the `rules` attribute.

<span style="font-size: 12px">`rules` is an array that accepts `functions`, `boolean`, and `string`. Functions pass an input value as an argument and must return either `true` / `false` or a `string` containing an error message. The input field will enter an error state if a function returns (or any value in the array contains) `false` or is a `string`.</span>

```html
<var-switch v-model="value" :rules="[(v) => v === true || 'Error！']"/>
```

## API

### Props

| prop | Description | Type | Default |
| ----- | -------------- | -------- | ---------- |
| `v-model` | Check status of Switch	| _any_ | `false` |
| `active-value` | The value when the switch is turned on	| _any_ | `true` |
| `inactive-value` | The value when the switch is turned off	| _any_ | `false` |
| `disabled` | Whether to disable switch| _boolean_ | `false` |
| `readonly` | Whether to readonly switch | _boolean_ | `false` |
| `loading` | Whether to show loading icon | _boolean_ | `false` |
| `ripple` | Whether to open ripple | _boolean_ | `true` |
| `color` | Background color when open | _string_ | `#2979ff` |
| `close-color` | Background color when close | _string_ | `#fff` |
| `loading-color` | Color of loading icon | _string_ | `#fff` |
| `size` | Size of switch | _string \| number_ | `20` |
| `rules`| Validation rules | _array_  | - |

### Events

| Event | Description | arguments |
| ----- | -------------- | -------- |
| `click` | Emitted when component is clicked | `event: Event` |
| `change` | Emitted when check status changed | `value: any` |

### Theme Variables
#### The following LESS variables can be overridden at build time to modify the theme style

| Variable | Default |
| --- | --- |
| `@switch-track-background` | `#fff` |
| `@switch-track-active-background` | `@color-primary` |
| `@switch-track-error-background` | `@color-danger` |
| `@switch-ripple-color` | `@color-primary` |
| `@switch-handle-background` | `#fff` |
| `@switch-handle-color` | `#fff` |
| `@switch-handle-active-background` | `@color-primary` |
| `@switch-handle-error-background` | `@color-danger` |
