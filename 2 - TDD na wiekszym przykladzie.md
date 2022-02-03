## TDD na większym przykładzie

Przejdziemy przez implementacje modułu **music-core**, o którym wspominaliśmy we wcześniejszym rozdziale gdy rozważaliśmy nad szkieletem rozwiązania.

Zaczniemy od najmniejszej składowej każdego instrumentu czyli dźwięku. Na początek modele. Cześć interfejsów przenieśliśmy z **NoteButtonComponent**, a teraz importujemy je tam z pliku **music-core/definitions.ts**

```ts
// music-core/definitions.ts
export const POSITIONS = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11] as const;
export const OCTAVES = [0, 1, 2, 3, 4, 5, 6, 7, 8] as const;
export const SHARP_NAMES = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"] as const;

export type NotePosition = typeof POSITIONS[number];
export type SharpNoteName = typeof SHARP_NAMES[number];
export type NoteOctave = typeof OCTAVES[number];

export interface Note {
  position: NotePosition; // czy jest to dźwięk C...B w postaci identyfikatorow 0...11
  octave: NoteOctave; // czy jest to oktawa 0...8
  id: string | number; // unikalny idenyfikator dźwięku
  name: string; // nazwa uproszczona czyli C, C#, D, D#...B
  halfEnharmonizedName: string // nazwa, w której nazwy dźwięków z # zamieniamy w według C# -> Db lub D# -> Eb
}
```

Następnie tworzymy szkielet klasy implementujący interfejs **Note**, tylko w celu zaspokojenia typów.

```ts
// music-core/guitarNote.ts
export class GuitarNote implements Note {
  name = "";
  halfEnharmonizedName = "";

  constructor(
    public position: NotePosition,
    public octave: NoteOctave,
    public id: NoteId
  ) {}
}
```

Teraz opisy testów. Proces sobie skracam nie zawsze musze przechodzić przez cały opisany w rozdziale 1. Wraz z nabraniemzynają się dziać automatycznie.

```ts

```
