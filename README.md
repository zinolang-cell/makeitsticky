<p align="center">
  <img src="assets/avolang-logo.png" alt="Avolang" width="220">
</p>

# Make It Sticky

`makeitsticky` is a Codex skill for building safe sticky, stacking, and scroll-driven sections on websites. It helps create visually compelling scroll transitions or repair existing implementations without clipping content, hiding it permanently, or making layouts unusable on shorter viewports.

## What the skill does

- inspects the existing page, section, and overflow structure;
- selects a safe sticky mode based on the content and available viewport height;
- treats sticky effects as progressive enhancement;
- separates sticky geometry from visual text fading;
- protects images, videos, SVGs, forms, and dynamic content;
- provides fallbacks for touch devices and `prefers-reduced-motion`;
- defines a browser QA matrix for scrolling, reverse scrolling, breakpoints, and anchor links.

## Typical use cases

- Headings should remain positioned directly below the navigation while scrolling.
- Consecutive sections should cover one another cleanly.
- Sticky content disappears too early, overlaps text, or gets clipped.
- Text should fade near the end of a section while media remains visible.
- An existing scroll animation only works at certain viewport sizes.

## Safety principle

The skill prioritizes content safety over animation. Content remains in the DOM and in the normal document flow. Unsafe or dynamic sections receive an opt-out, while the page remains fully visible and usable without JavaScript, on touch devices, and when reduced motion is enabled.

## Usage

After installation, invoke the skill in Codex with `$makeitsticky`, for example:

```text
Use $makeitsticky to review the sticky headings on this landing page and repair them safely.
```

The complete implementation and validation rules are documented in [`SKILL.md`](SKILL.md). UI metadata is stored in [`agents/openai.yaml`](agents/openai.yaml).

## Avolang

Developed by [Avolang](https://www.avolang.de) for robust, accessible, and maintainable web experiences.

Contact: [info@avolang.de](mailto:info@avolang.de)

## License

The skill code and documentation are available under the [MIT License](LICENSE). The Avolang name, Avolang logo, and all files in the `assets/` directory are expressly excluded and remain fully protected. These brand elements are governed by the separate [Avolang Brand Assets License](assets/LICENSE).

The public repository may be viewed and forked. Direct changes to the original repository can only be made by its owner or by GitHub accounts explicitly authorized by the owner.
