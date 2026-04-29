# Operating-mode `AskUserQuestion` shape

Used by SKILL.md §1.1a. Paste verbatim, adjust nothing — the option labels and descriptions are load-bearing (the user picks based on the cons line).

```js
AskUserQuestion({
  questions: [{
    header: "Run mode",
    question: "How involved do you want to be in this run? This affects how many decisions I ask you to make before we ship an artifact.",
    multiSelect: false,
    options: [
      {
        label: "Semi-auto (Recommended)",
        description: "I decide vibe, audience, scenario, dimensions, copy. You decide: output format (GIF/PNG/HTML) and one preview-and-iterate review before export. ~2 decisions. Pros: fast, keeps production-grade gate, keeps your taste in the loop on the things that matter most. Cons: you miss input on smaller creative calls."
      },
      {
        label: "Manual",
        description: "I ask you at every decision point — scenario pick, vibe confirmation, brief approval, preview iteration rounds, export ship-intent. I still make suggestions and recommendations at each step. Pros: max control, highest ceiling on quality. Cons: slow — 8–12 back-and-forths before an artifact."
      },
      {
        label: "Auto",
        description: "I make every decision and go straight to the exported artifact (GIF or PNG, my call). Pros: hands-off, 0 decisions, fastest path to a shippable draft. Cons: lower quality ceiling, more risk of missing your taste or the repo's real scope; expect to redirect after seeing the result."
      }
    ]
  }]
})
```

Default recommendation: **Semi-auto**. After the user picks, commit it to memory for the run (e.g. "Mode: Semi-auto") and reference at every subsequent decision point.
