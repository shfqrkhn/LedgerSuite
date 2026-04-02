## 2026-04-02 - [Sentinel Mode] - [Insight]
**Protocol:** Avoid innerHTML entirely to enforce strict defense-in-depth against XSS injection; rely solely on pure DOM APIs (`document.createElement`, `document.createTextNode`, `replaceChildren`).
