---
name: makeitsticky
description: Sichere Sticky-, Stacking- und Scroll-Sektionen für Websites entwickeln, prüfen und reparieren. Verwenden bei sticky sections, pinned headings, section stacking, Scroll-Übergängen, text-only End-Fades, stabil sichtbaren Bildern und Problemen wie überlagertem, abgeschnittenem beziehungsweise unsichtbarem Inhalt; geeignet für HTML/CSS/JavaScript sowie React-basierte Frontends und visuelle Browser-QA.
---

# Make It Sticky: sichere Sticky-Sektionen entwickeln

## Ziel

Sticky- und Stacking-Animationen als progressive Verbesserung umsetzen. Den visuellen Effekt deutlich machen, ohne Inhalt aus dem Dokumentfluss zu entfernen, Layouts zu beschädigen, Navigation zu verändern oder Inhalte bei einer ungewöhnlichen Viewport-Höhe dauerhaft zu verstecken.

Den Effekt immer als Zusammenspiel von drei Ebenen behandeln:

1. **Section cover:** Eine spätere Sektion deckt die frühere mit eigenem, deckendem Hintergrund zuverlässig ab.
2. **Sticky target:** Nur ein nachweislich geeignetes Element wird festgehalten: ganze kompakte Sektion, innerer Container oder Kopfbereich.
3. **End fade:** Nur das tatsächlich festgehaltene Element wird beim Verlassen seines Abschnitts ausgeblendet und beim Zurückscrollen wieder eingeblendet.

## Unverhandelbare Sicherheitsregeln

- Sämtliche Inhalte im DOM und im normalen Dokumentfluss belassen.
- Niemals eine ganze inhaltstragende Sektion am Ende per `display: none`, `opacity: 0` oder `visibility: hidden` verschwinden lassen.
- Nur ein Element ausblenden, dessen echtes Sticky-Verhalten zuvor im Browser bestätigt wurde.
- Sticky-Geometrie und Fade-Ziel voneinander trennen: Text darf ausblenden, Bilder, Videos, SVGs und Canvas-Inhalte nicht.
- Einen Opt-out pro Sektion vorsehen, beispielsweise `data-motion-stack="off"`.
- Opt-out-Sektionen weiterhin als deckende Ebene behandeln, wenn davor eine Sticky-Sektion liegt. Sonst kann der vorherige Inhalt durchscheinen oder darüberliegen.
- Formular-, FAQ-, Accordion-, Tabellen-, Iframe-, Slider-, Carousel- und rechtlich sensible Bereiche standardmäßig nicht automatisch sticky machen.
- Auf kleinen Viewports, Touch-Geräten und bei `prefers-reduced-motion: reduce` auf den normalen Dokumentfluss zurückfallen.
- Ohne JavaScript alle Inhalte sichtbar und bedienbar halten.
- Vorhandene Links, Pfade, IDs und Anker nicht für den Animationseffekt umbenennen.
- Vorhandene Nutzeränderungen und Seitenspezifika nicht durch eine globale Neustrukturierung überschreiben.

## Arbeitsablauf

### 1. Bestehende Struktur untersuchen

Vor Änderungen Folgendes ermitteln:

- Welche Seiten sollen Animationen erhalten und welche müssen ausgeschlossen bleiben?
- Welche direkten `section`-Kinder besitzt `main`?
- Welche Sektionen haben transparente Hintergründe?
- Welche Elemente sind bereits `position: sticky`, `overflow: hidden`, `overflow: clip`, transformiert oder animiert?
- Welche Sektionen enthalten Formulare, `details`, Tabellen, Medien, Accordions oder dynamische Höhen?
- Welche alten Reveal-Animationen setzen initial `opacity: 0`?
- Welche Navigation, Hash-Anker und extensionlosen Routen müssen unverändert funktionieren?

Nicht allein auf Klassennamen vertrauen. Die tatsächlichen berechneten Styles und Rechtecke im Browser prüfen.

### 2. Progressive Aktivierung einrichten

Den Effekt nur auf ausdrücklich aktivierten Seiten starten. Eine robuste Aktivierung:

- Pfad normalisieren: `.html` und abschließenden Slash entfernen, `/` erhalten.
- Impressum, Datenschutz, AGB und vergleichbare Rechtsseiten ausschließen.
- Eine Root-Klasse wie `.motion-enabled` erst per JavaScript setzen.
- Sticky stacking nur bei Desktop mit feinem Pointer aktivieren, beispielsweise:

```js
const desktop = window.matchMedia(
  '(min-width: 901px) and (hover: hover) and (pointer: fine)'
);
const reduceMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;
```

Bei Nichtübereinstimmung alle dynamischen Klassen und Inline-Custom-Properties sauber zurücksetzen.

### 3. Sektionen sicher schichten

`main` mit `position: relative` und `isolation: isolate` als lokale Stacking-Umgebung verwenden. Jede beteiligte Sektion benötigt:

- `position: relative`, solange sie nicht selbst sticky ist;
- eine eindeutige, aufsteigende `z-index`-Reihenfolge;
- einen deckenden Hintergrund;
- optional einen abgerundeten oberen Rand und Schatten beim Wechsel zwischen hellen und dunklen Flächen.

Einen scheinbar transparenten Sektionshintergrund vom nächsten deckenden Vorfahren ableiten. Die ursprüngliche berechnete Farbe als Custom Property setzen, statt pauschal überall dieselbe Farbe zu erzwingen.

```css
.motion-enabled main.motion-strong-stack {
  position: relative;
  isolation: isolate;
}

.motion-enabled main.motion-strong-stack > section.motion-stack-cover {
  position: relative;
  z-index: var(--motion-stack-z, 1);
  background-color: var(--motion-stack-bg, #07111f);
}
```

Keine negative Marge verwenden, um Sticky-Übergänge zu simulieren. Das verändert den Dokumentfluss und war eine Hauptquelle für abgeschnittene oder übersprungene Inhalte.

### 4. Den passenden Sticky-Modus wählen

Die verfügbare Höhe aus Viewport minus Navigation und Sicherheitsabstand berechnen. Anschließend genau einen Modus auswählen.

#### Ganze Sektion

Nur verwenden, wenn die Sektion vollständig in die verfügbare Höhe passt und ihr `scrollHeight` nicht größer als diese Höhe ist. Die ganze Sektion darf keine Formulare, `details`, Iframes oder dynamisch expandierende Inhalte enthalten.

```css
.motion-stack-panel {
  position: sticky;
  top: var(--motion-stack-top, 72px);
  min-height: calc(100svh - var(--motion-stack-top, 72px));
  box-sizing: border-box;
}
```

Ganze Sticky-Sektionen nicht selbst am Ende ausblenden. Eine nachfolgende deckende Sektion übernimmt das visuelle Ablösen.

#### Innerer Container

Bei einer hohen Sektion einen einzelnen direkten Container wie `*-inner`, `*-content`, `*-container` oder `*-wrap` auswählen. Nur aktivieren, wenn:

- der Container vollständig in die verfügbare Höhe passt;
- die Sektion deutlich höher als der Container ist;
- mindestens etwa 80 Pixel sichere Sticky-Reise vorhanden sind;
- kein blockierender Overflow-Vorfahre das echte Sticky-Verhalten verhindert.

#### Kopfbereich

Wenn nur die Einleitung festgehalten werden soll, einen klaren Header-Container bevorzugen. Der Header muss deutlich kleiner als die verfügbare Höhe sein und genügend folgenden Inhalt innerhalb derselben Sektion besitzen.

Eine einzelne große Überschrift nicht automatisch sticky machen, wenn Fließtext unmittelbar darunter weiterläuft. Dabei kann die Überschrift den Text überdecken. Den kompletten semantischen Kopfcontainer verwenden oder die Sektion vom Sticky-Verhalten ausnehmen.

### Bündiger Sticky-Kopf mit deckender Fläche

Jeder aktivierte Sticky-Kopf muss direkt an der Unterkante der festen Navigation beginnen. Kein zusätzlicher vertikaler Abstand zwischen Navigation und Sticky-Kopf: In diesem Zwischenraum könnte vorbeiscrollender Text sichtbar werden. Der Kopf selbst erhält den tatsächlich berechneten, deckenden Hintergrund seiner Sektion und eine untere Innenkante, die bis zur Text-Unterkante reicht. So wirkt der Kopf beim Überfahren wie eine saubere Maske: Der nachfolgende Text bleibt im DOM, wird im Bereich des Kopfes aber vollständig verdeckt und beim Hochscrollen sofort wieder sichtbar.

```css
.motion-stack-header-panel {
  position: sticky;
  top: var(--motion-stack-top, 72px); /* bündig unter der Navigation */
  z-index: 3;
  padding-top: 0;
  padding-bottom: 22px;
  background-color: var(--motion-stack-bg, #07111f);
  box-shadow: 0 24px 34px var(--motion-stack-bg, #07111f);
}
```

Die Regel zentral anwenden, statt individuelle `top: calc(... + Abstand)`-Werte je Sektion zu pflegen. Für helle und dunkle Sektionen immer die jeweilige berechnete Hintergrundfarbe verwenden; ein transparenter Sticky-Kopf ist kein zulässiger Ersatz für die deckende Fläche.

### 5. Unsichere Sektionen ausschließen

Sowohl Klassennamen als auch Inhalt prüfen. Typische Ausschlüsse:

```js
const unsafeClass = /faq|sources|article|legal|form|process|accordion|marquee|carousel|slider/i;
const unsafeContent = [
  'form', 'iframe', 'table', 'details',
  '[class*="accordion"]', '[class*="marquee"]',
  '[class*="carousel"]', '[class*="slider"]'
].join(',');
```

Nach der Erkennung eines unsicheren Bereichs **vor** jeder automatischen Header-Auswahl zurückkehren. Ein häufiger Fehler besteht darin, ganze und innere Panels korrekt auszuschließen, anschließend aber trotzdem den Header sticky zu machen.

Einen expliziten Opt-out höher priorisieren als alle Heuristiken:

```html
<section class="knowledge-section" data-motion-stack="off">
  ...
</section>
```

Die Sektion darf weiterhin `motion-stack-cover`, Hintergrund und `z-index` erhalten. Keine `motion-stack-panel`, `motion-stack-inner-panel` oder `motion-stack-header-panel` zuweisen.

## End-Fade korrekt umsetzen

### Warum die Sektionskante allein nicht genügt

Nicht nur `section.bottom - sticky.bottom` verwenden. Unterschiedliche Innenabstände und lange Inhaltsbereiche erzeugen einen variablen Restabstand. Dadurch kann ein Fade nie auslösen oder zu früh beginnen.

Ebenso wenig reicht die vorhandene Sticky-Klasse als Beweis. Ein Vorfahr mit `overflow: hidden` kann dazu führen, dass ein Element zwar `position: sticky` berechnet, im Viewport aber normal weiterscrollt. Ein blindes Fade würde dann normalen Inhalt unsichtbar machen.

### Echtes Festhalten nachweisen

Pro Ziel einen Laufzeitzustand speichern:

```js
{
  stickyElement,
  fadeElements,
  section,
  wasPinned: false,
  pinnedFrames: 0,
  lastTop: null,
  lastScrollY: window.scrollY
}
```

Das Element erst als tatsächlich sticky markieren, wenn:

- seine obere Viewport-Position dem berechneten Sticky-`top` entspricht;
- sich die Seite zwischen zwei Frames bewegt hat;
- die obere Position des Elements trotz Seitenbewegung stabil geblieben ist.

Damit wird echtes Festhalten von einem Element unterschieden, das nur zufällig die Sticky-Linie kreuzt.

```js
const rect = target.stickyElement.getBoundingClientRect();
const stickyTop = Number.parseFloat(
  getComputedStyle(target.stickyElement).top
) || 0;

const topIsStable = target.lastTop !== null &&
  Math.abs(rect.top - target.lastTop) < 0.75;
const pageMoved = Math.abs(scrollY - target.lastScrollY) > 0.75;
const isAtStickyTop = Math.abs(rect.top - stickyTop) < 1.5;

if (isAtStickyTop && topIsStable && pageMoved) {
  target.pinnedFrames += 1;
  if (target.pinnedFrames >= 1) target.wasPinned = true;
} else if (!isAtStickyTop) {
  target.pinnedFrames = 0;
}
```

### Sticky-Geometrie von Text-Fade trennen

Den Sticky-Container nicht automatisch selbst ausblenden. Ein Container kann Text und Medien gemeinsam enthalten. Stattdessen zwei Rollen speichern:

- `stickyElement`: Element, dessen echte Sticky-Position gemessen wird;
- `fadeElements`: ausschließlich medienfreie Textäste, auf die Filter und Hidden-Zustand angewendet werden.

Medien mindestens mit diesem Selektor schützen:

```js
const mediaSelector = 'img, picture, video, canvas, svg';
```

Textziele rekursiv sammeln. Einen medienfreien Container als Ganzes verwenden. Enthält er Medien, nur seine medienfreien Kinder mit tatsächlichem Text übernehmen und bei gemischten Kindern weiter absteigen.

```js
function collectTextFadeElements(container) {
  if (container.matches(mediaSelector)) return [];

  if (!container.querySelector(mediaSelector)) {
    return container.textContent.trim() ? [container] : [];
  }

  return Array.from(container.children).flatMap((child) => {
    if (child.matches(mediaSelector) || !child.textContent.trim()) return [];
    return collectTextFadeElements(child);
  });
}
```

Damit kann beispielsweise der Text neben einem Video verschwinden, während das Video sichtbar bleibt. Auch ein SVG-Icon darf nicht versehentlich verschwinden, nur weil es gemeinsam mit Text in einer Statistikspalte liegt.

Jedes Fade-Ziel mit einer eigenen Klasse wie `.motion-stack-text-fade-target` markieren. Beim Reset Klasse, Hidden-Zustand und Fade-Property von allen `fadeElements` entfernen.

### Kurz vor der Sektionsgrenze ausblenden

Nach bestätigtem Festhalten zwei Signale kombinieren:

1. die reale Bewegung oberhalb der Sticky-Linie;
2. die von unten herannahende Sektionsgrenze.

Nicht ausschließlich auf das tatsächliche Verlassen warten. Am Seitenende kann weiterer Scrollweg fehlen, sodass eine Überschrift nur teilweise verblasst über dem Footer stehen bleibt. Den Fade deshalb bereits vor der unteren Grenze beginnen und mit einem zur freien Viewport-Fläche passenden Sicherheitsabstand vollständig abschließen.

```js
const fadeDistance = Math.max(96, Math.min(160, innerHeight * 0.18));

const distancePastEnd = Math.max(0, stickyTop - rect.top);
const departureFade = Math.max(
  0,
  Math.min(1, 1 - distancePastEnd / fadeDistance)
);

const sectionRect = target.section.getBoundingClientRect();
const freeSpaceBelow = Math.max(0, innerHeight - rect.bottom);
const earlyBoundaryGap = Math.max(
  56,
  Math.min(190, freeSpaceBelow * 0.28)
);
const distanceToBoundary = sectionRect.bottom - rect.bottom;
const boundaryFade = Math.max(
  0,
  Math.min(1, (distanceToBoundary - earlyBoundaryGap) / fadeDistance)
);

const fade = target.wasPinned
  ? Math.min(departureFade, boundaryFade)
  : 1;

target.fadeElements.forEach((fadeElement) => {
  fadeElement.style.setProperty(
    '--motion-stack-end-fade', fade.toFixed(3)
  );
  fadeElement.classList.toggle(
    'motion-stack-end-hidden', fade <= 0.015
  );
});
```

Das Fade über eine eigene visuelle Eigenschaft komponieren. Wenn vorhandene Reveal-Regeln bereits `opacity` steuern, nicht dieselbe Property überschreiben. `filter: opacity()` funktioniert für neutrale Container; bei bereits vorhandenen Filtern einen zusätzlichen Wrapper einsetzen.

```css
.motion-stack-text-fade-target {
  filter: opacity(var(--motion-stack-end-fade, 1));
  transition: filter 0.16s linear;
}

.motion-stack-end-hidden {
  visibility: hidden;
  pointer-events: none;
}
```

Beim Hochscrollen den Fade-Wert automatisch wieder auf `1` bringen und die Hidden-Klasse entfernen. Bei Reset oder Breakpoint-Wechsel alle Custom Properties und Hidden-Klassen entfernen.

Die Vorlaufwerte als responsive Sicherheitsbereich behandeln, nicht als absolute Designkonstante. Bei typischen Desktop-Viewports sollte ein kurzer Textkopf spätestens verschwunden sein, wenn die untere Sektionsgrenze ungefähr die unteren 250 bis 320 Pixel des Viewports erreicht. Große Sticky-Inhalte benötigen einen kleineren Abstand unter ihrem eigenen sichtbaren Rand.

### Scroll-Arbeit begrenzen

Pro Scroll-Frame höchstens eine Messung ausführen:

```js
let fadeFramePending = false;

function scheduleFadeUpdate() {
  if (fadeFramePending) return;
  fadeFramePending = true;
  requestAnimationFrame(() => {
    updateStickyFades();
    fadeFramePending = false;
  });
}

addEventListener('scroll', scheduleFadeUpdate, { passive: true });
```

Layoutmessungen zuerst sammeln und danach Styles setzen, wenn viele Ziele existieren, um Layout-Thrashing zu minimieren.

## Reset und Neuberechnung

Vor jeder erneuten Klassifizierung vollständig zurücksetzen:

- Hauptklasse für starkes Stacking entfernen;
- Sticky-, Cover-, Farbwechsel- und Hidden-Klassen entfernen;
- dynamische `z-index`-, Hintergrund- und Fade-Properties entfernen;
- registrierte Fade-Ziele leeren;
- anschließend anhand der aktuellen Viewport-Größe neu entscheiden.

Neu berechnen bei:

- `resize` mit kurzem Debounce;
- `load`;
- Abschluss von `document.fonts.ready`;
- Größenänderungen relevanter Sektionen über `ResizeObserver`.

Einen `ResizeObserver` nicht durch eigene Styleänderungen in eine Endlosschleife bringen. Änderungen bündeln und die Klassifizierung idempotent halten.

## Häufige Fehlerbilder und Reparaturen

### Überschrift überdeckt Fließtext

**Ursache:** Eine einzelne Überschrift wurde sticky, der restliche Inhalt scrollt dahinter.

**Reparatur:** Semantischen Header-Container verwenden oder `data-motion-stack="off"` für die Sektion setzen. Im Browser `getComputedStyle(header).position` kontrollieren.

### Inhalt erscheint gar nicht

**Ursache:** Reveal-Element startet mit `opacity: 0`, aber Observer oder JavaScript läuft nicht; alternativ wurde ein ganzes Panel ausgeblendet.

**Reparatur:** Ohne `.js` beziehungsweise ohne `.motion-enabled` sichtbare Defaults verwenden. Reduced-Motion-Fallback erzwingen. Niemals die komplette Inhaltssektion als End-Fade-Ziel registrieren.

### Sektion ist abgeschnitten

**Ursache:** Höhe größer als verfügbarer Viewport, `overflow: hidden`, falsches `min-height`, negative Margen oder Sticky-Kind außerhalb seines sicheren Containers.

**Reparatur:** Tatsächliche `scrollHeight` gegen verfügbare Höhe prüfen. Zu große Sektionen normal scrollen lassen. Overflow nur dort einsetzen, wo keine Sticky-Nachfahren existieren.

### Layout funktioniert auf einer Seite, bricht auf einer anderen

**Ursache:** Zu allgemeine Selektoren wählen in unterschiedlich strukturierten Seiten verschiedene Elemente aus.

**Reparatur:** Heuristiken mit expliziten Opt-outs kombinieren. Alle Zielseiten einzeln im Browser prüfen. Seiteninhalt nicht anhand eines einzigen Templates annehmen.

### FAQ-Kopf bleibt kleben

**Ursache:** Die Sektion wird wegen `details` als unsicher erkannt, der Code wählt danach trotzdem noch einen Header aus.

**Reparatur:** Bei `unsafe === true` vor der Header-Heuristik abbrechen oder die konkrete FAQ-Sektion explizit ausschließen.

### Nicht-sticky Inhalt wird ausgeblendet

**Ursache:** Die Klasse `position: sticky` wurde mit echtem Festhalten verwechselt; ein Overflow-Vorfahr verhindert das Pinnen.

**Reparatur:** `wasPinned` nur nach stabiler Viewport-Position bei gleichzeitig verändertem `scrollY` setzen. Ohne diesen Nachweis Fade immer auf `1` lassen.

### Bilder oder Videos verschwinden gemeinsam mit Text

**Ursache:** Der Filter wurde auf den gesamten Sticky-Wrapper gesetzt, obwohl dieser neben Text auch `img`, `picture`, `video`, `canvas` oder `svg` enthält.

**Reparatur:** Sticky-Messung auf dem Wrapper belassen, den Fade jedoch ausschließlich auf rekursiv ermittelte medienfreie Textäste anwenden. Im Browser sicherstellen, dass kein Medium einen Vorfahren mit `.motion-stack-text-fade-target` besitzt.

### Sticky-Überschrift bleibt am Seitenende teilweise sichtbar

**Ursache:** Der Fade beginnt erst, nachdem das Sticky-Element seine obere Halteposition bereits verlassen hat. Vor dem Footer steht nicht immer genug weiterer Scrollweg zur Verfügung, um den Fade abzuschließen.

**Reparatur:** Den Departure-Fade mit einem frühen Boundary-Fade kombinieren und jeweils den kleineren Wert anwenden. Dadurch ist der Text bereits vor der herannahenden Sektionsgrenze vollständig verborgen und erscheint beim Hochscrollen wieder.

### Übergang zeigt die vorherige Sektion durch

**Ursache:** Die nächste Sektion ist transparent oder besitzt keinen höheren lokalen `z-index`.

**Reparatur:** Deckenden berechneten Hintergrund, aufsteigenden `z-index` und `isolation: isolate` verwenden. Auch Opt-out-Sektionen als Cover behalten.

## Responsive und barrierefreie Rückfälle

Unterhalb des Desktop-Breakpoints und bei grobem Pointer alle Sticky-Eigenschaften neutralisieren:

```css
@media (max-width: 900px), (hover: none), (pointer: coarse) {
  .motion-stack-cover,
  .motion-stack-panel,
  .motion-stack-inner-panel,
  .motion-stack-header-panel {
    position: relative;
    top: auto;
    min-height: 0;
    z-index: auto;
    box-shadow: none;
    filter: none;
    visibility: visible;
  }
  .motion-stack-text-fade-target {
    filter: none;
    visibility: visible;
    transition: none;
  }
}
```

Dieselbe Neutralisierung für `prefers-reduced-motion: reduce` anwenden. Scroll-Verhalten nicht erzwingen und keine Inhalte aufgrund einer Animationspräferenz entfernen.

Fokusreihenfolge, Skip-Links, Ankerziele und Tastaturbedienung im DOM-Fluss belassen. Ein ausgeblendetes Sticky-Duplikat darf keine fokussierbaren Elemente enthalten; falls doch, beim Hidden-Zustand Interaktion sicher deaktivieren oder die Architektur ändern.

## Browser-QA

Nach jeder Änderung die tatsächliche Seite im Browser prüfen. Statische Codekontrolle allein reicht für Sticky-Verhalten nicht aus.

### Mindestmatrix

- Alle aktivierten Hauptseiten aufrufen.
- Alle Navigationslinks und extensionlosen Routen öffnen.
- Desktop mit kurzer Höhe, normaler Höhe und breitem Viewport testen.
- Grenze knapp oberhalb und unterhalb des Desktop-Breakpoints testen.
- Touch/mobile und Reduced Motion prüfen.
- Direktaufruf über Hash-Anker und Reload in Seitenmitte prüfen.
- Vorwärts bis zum Sektionsende und anschließend rückwärts scrollen.
- FAQ öffnen, Formulare bedienen und dynamische Inhalte aufklappen.
- Sicherstellen, dass Impressum, Datenschutz und AGB unverändert bleiben, wenn sie ausgeschlossen sind.

### Technische Assertions

Für jedes Ziel mindestens prüfen:

```js
const sticky = document.querySelector('.motion-stack-header-panel');
const fadeTarget = sticky.matches('.motion-stack-text-fade-target')
  ? sticky
  : sticky.querySelector('.motion-stack-text-fade-target');
const section = sticky.closest('section');

({
  targetPosition: getComputedStyle(sticky).position,
  targetTop: sticky.getBoundingClientRect().top,
  sectionBottom: section.getBoundingClientRect().bottom,
  fade: fadeTarget.style.getPropertyValue('--motion-stack-end-fade'),
  hidden: fadeTarget.classList.contains('motion-stack-end-hidden'),
  mediaInsideFade: Boolean(
    fadeTarget.querySelector('img, picture, video, canvas, svg')
  )
});
```

Folgende Zustände verifizieren:

1. **Vor Sticky:** Fade `1`, sichtbar, Inhalt im normalen Fluss.
2. **Während Sticky:** Position bleibt trotz verändertem `scrollY` an der Sticky-Linie, Fade `1`.
3. **Kurz vor dem Ende:** Fade erreicht bereits vor der unteren Sektionsgrenze `0`, nur Sticky-Textziel hidden.
4. **Zurückscrollen:** Fade wieder `1`, Hidden-Klasse entfernt.
5. **Falscher Sticky-Kandidat:** Auch nach Passieren der oberen Kante Fade `1` und sichtbar.
6. **Gemischter Container:** Textziel darf ausblenden; Medien bleiben sichtbar und liegen außerhalb jedes Fade-Ziels.

Zusätzlich Rechtecke vergleichen: Überschrift-Unterkante muss vor Text-Oberkante liegen, Text-Unterkante vor dem nächsten Inhaltsblock. Bei wichtigen Sektionen automatisierte Screenshots erzeugen und visuell auf Überlagerung, Clipping sowie plötzlich leere Flächen prüfen.

## Entscheidung: reparieren oder Opt-out setzen

Die Heuristik reparieren, wenn dasselbe strukturelle Problem auf mehreren Seiten auftritt. Einen gezielten Opt-out setzen, wenn die betreffende Sektion einzigartig ist, dynamische Inhalte enthält oder der Sticky-Effekt dort keinen klaren Mehrwert bietet.

Im Zweifel Inhaltssicherheit priorisieren: Ein normal scrollender Abschnitt ist akzeptabel; ein beeindruckender Effekt mit fehlendem Inhalt ist es nicht.
