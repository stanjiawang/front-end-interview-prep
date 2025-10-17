# 🎨 Bootstrap & React-Bootstrap Interview Preparation Guide

A comprehensive guide to **Bootstrap** and **React-Bootstrap** for interviews.  
Includes fundamentals, theming, React integration, and common Q&A.

---

## 📑 Table of Contents
- [1. What is Bootstrap?](#1-what-is-bootstrap)
- [2. Bootstrap Fundamentals](#2-bootstrap-fundamentals)
  - [2.1 Grid System](#21-grid-system)
  - [2.2 Utilities](#22-utilities)
  - [2.3 Components](#23-components)
  - [2.4 JavaScript Plugins](#24-javascript-plugins)
- [3. What is React-Bootstrap?](#3-what-is-react-bootstrap)
- [4. Installation & Setup](#4-installation--setup)
- [5. Core Concepts of React-Bootstrap](#5-core-concepts-of-react-bootstrap)
- [6. Common React-Bootstrap Components](#6-common-react-bootstrap-components)
- [7. Theming & Customization](#7-theming--customization)
- [8. Accessibility](#8-accessibility)
- [9. Integration with React Ecosystem](#9-integration-with-react-ecosystem)
- [10. Best Practices](#10-best-practices)
- [11. Common Interview Questions](#11-common-interview-questions)

---

## 1. What is Bootstrap?

Bootstrap is the **most popular CSS framework** for building responsive, mobile-first websites.  
- Provides CSS and JS components (grid, utilities, modals, forms).  
- Supports customization via Sass variables.  
- Current version: **Bootstrap 5** (removed jQuery dependency).  

---

## 2. Bootstrap Fundamentals

### 2.1 Grid System
- Based on a **12-column layout**.  
- Breakpoints: `xs`, `sm`, `md`, `lg`, `xl`, `xxl`.  

Example:
```html
<div class="container">
  <div class="row">
    <div class="col-md-4">Col 1</div>
    <div class="col-md-8">Col 2</div>
  </div>
</div>
```

### 2.2 Utilities
Quick utility classes for spacing, colors, typography, flex, etc.  

```html
<p class="text-center text-primary fw-bold m-3">Hello Bootstrap</p>
```

### 2.3 Components
Prebuilt UI elements:
- Buttons (`btn btn-primary`)  
- Navbar  
- Forms  
- Cards  
- Alerts  

### 2.4 JavaScript Plugins
Interactive components (modals, dropdowns, tooltips).  
- Bootstrap 5 uses **Vanilla JS** (no jQuery required).  

---

## 3. What is React-Bootstrap?

React-Bootstrap is a **React-specific reimplementation** of Bootstrap’s components.  
- Written in React (no jQuery).  
- Uses **props instead of classes**.  
- Provides accessible, React-friendly components.  
- Syncs with Bootstrap’s design system.  

---

## 4. Installation & Setup

```bash
npm install react-bootstrap bootstrap
```

Import Bootstrap CSS:
```js
import 'bootstrap/dist/css/bootstrap.min.css';
```

Use components:
```jsx
import Button from 'react-bootstrap/Button';

function App() {
  return <Button variant="primary">Click Me</Button>;
}
```

---

## 5. Core Concepts of React-Bootstrap

- Replace `class` with **props** (`variant`, `size`).  
- Uses **declarative API** consistent with React.  
- Components are accessible by default.  
- Grid system provided via `<Container>`, `<Row>`, `<Col>`.  

---

## 6. Common React-Bootstrap Components

### Buttons
```jsx
<Button variant="primary">Primary</Button>
<Button variant="outline-success">Success</Button>
```

### Forms
```jsx
<Form>
  <Form.Group>
    <Form.Label>Email</Form.Label>
    <Form.Control type="email" placeholder="Enter email" />
  </Form.Group>
</Form>
```

### Navbar
```jsx
<Navbar bg="dark" variant="dark" expand="lg">
  <Navbar.Brand href="#">MyApp</Navbar.Brand>
  <Nav className="me-auto">
    <Nav.Link href="#home">Home</Nav.Link>
    <Nav.Link href="#features">Features</Nav.Link>
  </Nav>
</Navbar>
```

### Modal
```jsx
<Modal show={show} onHide={handleClose}>
  <Modal.Header closeButton>
    <Modal.Title>Dialog</Modal.Title>
  </Modal.Header>
  <Modal.Body>Content goes here</Modal.Body>
  <Modal.Footer>
    <Button onClick={handleClose}>Close</Button>
  </Modal.Footer>
</Modal>
```

---

## 7. Theming & Customization

- Use `variant` prop (e.g., `variant="danger"`).  
- Override Bootstrap variables via **Sass**.  
- Add custom CSS.  
- Use **utility classes** when needed.  

---

## 8. Accessibility

- ARIA roles are built-in.  
- Keyboard navigation supported.  
- `<Form.Label>` correctly associates with `<Form.Control>`.  

---

## 9. Integration with React Ecosystem

- **React Router**:  
```jsx
<Nav.Link as={Link} to="/about">About</Nav.Link>
```

- **Formik/React Hook Form**: Works with `<Form.Control>`.  
- **Icons**: `react-icons` or `bootstrap-icons-react`.  

---

## 10. Best Practices

- Import components individually to reduce bundle size.  
- Use props API over manual classes.  
- Keep forms controlled.  
- Use `<Container fluid>` for full-width layouts.  

---

## 11. Common Interview Questions

### Q1. What is the difference between Bootstrap and React-Bootstrap?  
**A:** Bootstrap is a CSS + JS framework, while React-Bootstrap reimplements its components as React components without jQuery.

---

### Q2. How does React-Bootstrap handle theming?  
**A:** Through `variant` props, Sass variable overrides, and utility classes.

---

### Q3. How do you use the grid system in React-Bootstrap?  
**A:** Use `<Container>`, `<Row>`, `<Col>` with breakpoint props (`xs`, `md`, etc.).

---

### Q4. How do you integrate React Router with React-Bootstrap?  
**A:** Use the `as` prop to render `Link` inside components.

---

### Q5. How do modals work in React-Bootstrap?  
**A:** Controlled via `show` and `onHide` props.

---

### Q6. Why choose React-Bootstrap over plain Bootstrap?  
**A:** It’s React-first, declarative, more accessible, and avoids jQuery.

---

### Q7. How to optimize bundle size with React-Bootstrap?  
**A:** Import individual components instead of the whole library.

---

✍️ **Pro Tip for interviews:** Emphasize how React-Bootstrap provides a **React-native API** for Bootstrap styling, with accessibility and declarative benefits.
