## React Best Practices

### Directory Structure

- 1 exported per file
  - Filenames matching component names
- consistent organization

### DRY

- Don't repeat your code
  - Refactor repeated UI into helper components
  - Refactor repeated hooks into custom hooks

### Consistent CSS

- Avoid mixing methods for including CSS
  - import "component.css" at the top of each component module
    - This is actually specific to Webpack and create-react-app uses Webpack under the hood
    - But may not work if not using Webpack
  - Inline style attributes
  - Global or page level stylesheets
  - CSS in JS libraries

### Keep HTML Semantic

- Use the correct grouping tags, not just divs
- Focus on accessibility (alt attributes, aria, etc.)
- Avoid extra divs with fragments

### Miscellaneous Tips

- Follow a style guide for consistency
- Stick to either functional components or class-based components
- Minimize prop-drilling, use Contexts or state management libraries
- Avoid imperative code such as `useImperativeHandle` as React is declarative
- Write self-documenting code
