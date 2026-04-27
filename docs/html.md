---
title: HTMLData Standard
icon: lucide/compass
---

# Indelible's HTMLData Standard

Use the Indelible HTMLData Standard to embed structured quote and article authorship attestation metadata directly into your web pages.

At the top level of your article, whether it be an `<article>`, `<main>`, or even `<body>` tag, you must include the **`data-indelible="{}"`**[^1] attribute to indicate that the content within is part of an attestation.

[^1]: The `data-indelible` attribute must contain a JSON object as specified in [the Indelible Standard](thestandard.md#attestations).

## To include or not to include

Indelible's HTMLData Standard is designed to be flexible, so you can choose to mark text as part of an attestation inclusively or exclusively.

### Inclusive Marking
By marking specific elements (e.g., `<p>`, `<h1>`, `<blockquote>`, etc.) with the **`data-indelible-include`** attribute, you indicate that only those marked elements are part of the attestation. This allows you to selectively attest specific sections of your content, such as quotes or key paragraphs, while leaving other parts of the article, like advertisements and captions, outside the attestation.
!!! note
    When using inclusive marking, any content that is not explicitly marked with `data-indelible-include` will be excluded from the attestation. This means that if you forget to mark a section of your article, it will not be included in the attestation, which could lead to incomplete attestations.

### Exclusive Marking
By marking a parent element (e.g., `<article>`, `<main>`, or `<body>`) with the **`data-indelible`** attribute without manually specifying any `data-indelible-include` tags, you indicate that all content within that element is part of the attestation. This is the most common approach and is ideal for articles where the entire content is being attested. To exclude specific elements from the attestation, you can use the **`data-indelible-exclude`** attribute on those elements.