---
---

# Micro-Frontends

Micro-Frontends (MFE) extend the microservices concept to the frontend, breaking down a monolith into independent, loosely coupled units that can be developed, tested, and deployed by autonomous teams.

## **1. Integration Patterns**

### **Build-Time Integration**
*   **Mechanism**: Published as `npm` packages and integrated during the container's build process.
*   **Pros**: Traditional dependency management; easiest for versioning and tree-shaking.
*   **Cons**: **Lockstep releases** (changing an MFE requires rebuilding the container); creates a hidden monolith at the build level.

### **Run-Time Integration**
The container application fetches MFEs dynamically in the browser.

*   **iFrames**:
    *   **Pros**: Strongest isolation (CSS, JS global scope).
    *   **Cons**: Massive performance overhead; difficult deep-linking; hard to make responsive; "yuck" factor.
*   **Web Components (Custom Elements)**:
    *   **Pros**: "The DOM is the API." Technology agnostic (React can talk to Vue via Custom Elements).
    *   **Cons**: Requires polyfills for older browsers; can be complex to manage lifecycle across frameworks.
*   **Module Federation (Webpack 5+)**:
    *   **Pros**: The modern standard. Allows MFEs to share dependencies (e.g., only one instance of React) while loading remote code as if it were local.
    *   **Cons**: Specific to build tools (Webpack/Vite); requires careful configuration of `shared` scopes.

### **Server-Side Integration**
*   **SSI (Server Side Includes) / ESI (Edge Side Includes)**: Nginx or CDN-level composition.
*   **Pros**: Best for SEO and "Initial Meaningful Paint"; low browser overhead.
*   **Cons**: Slowest fragment dictates page response time; hard to handle complex client-side interactions.

---

## **2. Communication between MFEs**

*   **Custom Events**: Using `window.dispatchEvent(new CustomEvent('mfe:action'))`.
    *   *Best practice*: Decoupled and follows native browser patterns.
*   **Window/Global Object**: Attaching a shared message bus or state to `window`.
    *   *Risk*: Namespace collisions; hard to debug "spaghetti" events.
*   **Address Bar (URL)**: Passing state via query parameters or route paths.
    *   *Benefit*: Essential for persistence and shareable links.

---

## **3. Shared State & Dependency Management**

*   **Preventing Bloat**: Use **Shared Vendors** in Module Federation to ensure common libraries (React, Lodash, etc.) aren't downloaded multiple times.
*   **Design Systems**: A shared, versioned UI library is the "glue" that prevents visual fragmentation.
*   **Shared State**: MFEs should **rarely share state** (e.g., Redux stores). If needed, use a thin "Shell" layer to pass down auth tokens or global settings.

---

## **4. When NOT to use Micro-Frontends**

*   **Small Teams**: If you only have one or two frontend teams, the operational overhead (CI/CD, governance) outweighs the benefits.
*   **Low Complexity**: If the application is a simple CRUD or a single-domain product.
*   **Performance-Critical Apps**: MFEs introduce multiple network requests and potential JS overhead that can degrade mobile performance.
*   **Lack of Automation**: If you don't have robust CI/CD pipelines, managing 10+ independent deployments will be a nightmare.

---

## **Summary for Study**

*   **Core Goal**: Independent deliverability and autonomous teams.
*   **Integration**: **Module Federation** is the current industry preference for SPAs; **Web Components** for technology-agnostic interoperability.
*   **Communication**: Favor **Custom Events** and **URL state** over global variables.
*   **Avoid**: "Micro-Frontend Anarchy" (using 5 different frameworks just because you can).
*   **Architecture Debt**: MFEs trade technical simplicity for organizational scalability.

