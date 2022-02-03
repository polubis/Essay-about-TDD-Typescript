## TDD na większym przykładzie

Przejdziemy przez implementacje fragmentu modułu **music-core/guitarNote**, o którym wspominaliśmy we wcześniejszym rozdziale gdy rozważaliśmy nad szkieletem rozwiązania. Opiszemy tylko implementację tego modułu ze względu na złożoność całego rozwiązania. Reszte można zobaczyć sobie w linku z rozdziału 1.

Zaczniemy od najmniejszej składowej każdego instrumentu czyli dźwięku. Na początek modele. Cześć interfejsów przenieśliśmy z **NoteButtonComponent**, a teraz importujemy je tam z pliku **music-core/definitions.ts**

```ts
// music-core/definitions.ts
export const POSITIONS = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11] as const;
export const OCTAVES = [0, 1, 2, 3, 4, 5, 6, 7, 8] as const;
export const SHARP_NAMES = [
  "C",
  "C#",
  "D",
  "D#",
  "E",
  "F",
  "F#",
  "G",
  "G#",
  "A",
  "A#",
  "B",
] as const;

export type NotePosition = typeof POSITIONS[number];
export type SharpNoteName = typeof SHARP_NAMES[number];
export type NoteOctave = typeof OCTAVES[number];
export enum NoteNotationSymbol {
  Sharp = "#",
  Bmoll = "b",
}

export type NoteId = string | number;
export interface Note {
  position: NotePosition;
  octave: NoteOctave;
  id: NoteId;
  name: string;
  halfEnharmonizedName: string;
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

Teraz opisy testów. Proces sobie skracam nie zawsze musze przechodzić przez cały opisany w rozdziale 1. Wraz z nabraniem wprawy pewne rzeczy zaczynają się dziać automatycznie.

```ts
// music-core/guitarNote.test.ts
import { GuitarNote } from "./guitarNote";

describe("GuitarNote", () => {
  it("assigns given properties", () => {
    const note = new GuitarNote(0, 0, 0);
    expect(note.position).toBe(0);
    expect(note.octave).toBe(0);
    expect(note.id).toBe(0);
  });

  it("gets not enharmonized name", () => {
    expect(new GuitarNote(0, 0, 0).name).toBe("C");
    expect(new GuitarNote(1, 0, 0).name).toBe("C#");
    expect(new GuitarNote(3, 0, 0).name).toBe("D#");
    expect(new GuitarNote(6, 0, 0).name).toBe("F#");
    expect(new GuitarNote(8, 0, 0).name).toBe("G#");
    expect(new GuitarNote(10, 0, 0).name).toBe("A#");
    expect(new GuitarNote(11, 0, 0).name).toBe("B");
  });

  it("gets half enharmonized name", () => {
    expect(new GuitarNote(0, 0, 0).halfEnharmonizedName).toBe("C");
    expect(new GuitarNote(1, 0, 0).halfEnharmonizedName).toBe("Db");
    expect(new GuitarNote(3, 0, 0).halfEnharmonizedName).toBe("Eb");
    expect(new GuitarNote(6, 0, 0).halfEnharmonizedName).toBe("Gb");
    expect(new GuitarNote(8, 0, 0).halfEnharmonizedName).toBe("Ab");
    expect(new GuitarNote(10, 0, 0).halfEnharmonizedName).toBe("Bb");
    expect(new GuitarNote(11, 0, 0).halfEnharmonizedName).toBe("B");
  });
});
```

Oczywiście testy wszystkie są czerwone. Więc musimy dostosować implementacje.

```ts
import {
  Note,
  NotePosition,
  NoteOctave,
  NoteId,
  SHARP_NAMES,
} from "./definitions";

export class GuitarNote implements Note {
  name: string;
  halfEnharmonizedName: string;

  constructor(
    public position: NotePosition,
    public octave: NoteOctave,
    public id: NoteId
  ) {
    this.name = SHARP_NAMES[position];

    if (this.name.includes("#")) {
      let positionAcc = position;
      positionAcc =
        position + 1 >= SHARP_NAMES.length
          ? 0
          : ((position + 1) as NotePosition);
      this.halfEnharmonizedName = SHARP_NAMES[positionAcc] + 'b';
    } else {
      this.halfEnharmonizedName = this.name;
    }
  }
}
```

![image](https://user-images.githubusercontent.com/22937810/152332402-c2e2c8df-6677-4941-85d5-c5474c1169f7.png)

Mamy zielone światło więc teraz czas na refaktor. Kod, który powstał jest ciężki do zrozumienia. Mamy tam doczynienia z **magic numbers**, **magic strings** oraz operacjami w konstruktorze bez żadnej funkcji. 

```ts
// utils.ts
import { NotePosition, POSITIONS } from "./definitions";

export const getNextPosition = (position: NotePosition): NotePosition => {
  const nextPosition = position + 1;
  return nextPosition === POSITIONS.length ? 0 : (nextPosition as NotePosition);
};


// guitarNote.ts
import {
  Note,
  NotePosition,
  NoteOctave,
  NoteId,
  SHARP_NAMES,
  NoteNotationSymbol,
} from "./definitions";
import { getNextPosition } from "./utils";

export class GuitarNote implements Note {
  name: string;
  halfEnharmonizedName: string;

  constructor(
    public position: NotePosition,
    public octave: NoteOctave,
    public id: NoteId
  ) {
    this.name = SHARP_NAMES[position];
    this.halfEnharmonizedName = this._getHalfEnharmonizedName(
      this.name,
      position
    );
  }

  private _getHalfEnharmonizedName = (
    name: string,
    position: NotePosition
  ): string => {
    if (!this.name.includes(NoteNotationSymbol.Sharp)) {
      return name;
    }

    return SHARP_NAMES[getNextPosition(position)] + NoteNotationSymbol.Bmoll;
  };
}
```

Warto zauważyć, że funkcja `getNextPosition()` została wrzucona w oddzielny plik. Może się ona przydać w innym module. Zatem warto do niej napisać testy. Tutaj idealny przykład tego, że od TDD czasami się odchodzi. Ten kod wcześniej był napisany w konstrukturze `guitarNote` i wykonując go w tym module mamy pewność, że działa - testy są zielone.

Jednak w innym module może nie działać - przykładowo przekazane zostaną inne argumenty. Więc do niego również powinniśmy napisać testy.

```ts
// utils.test.ts
import { getNextPosition } from "./utils";

describe("getNextPosition()", () => {
  it("gets next position", () => {
    expect(getNextPosition(0)).toBe(1);
  });

  it("when last position resets to first one", () => {
    expect(getNextPosition(11)).toBe(0);
  });
});
```

W dalszym ciągu jednak w aplikacji odnosimy się do **magic numbers**. Kiedy mamy na myśli jakiś dźwięk zawsze wcześniej musimy w głowie wykonać mapowanie. 
Dźwięk C to 0, dźwięk C# to 1. Poprawmy też to. Tutaj wprowadzimy pierwszy raz zmiany w interfejsach w momencie gdy mamy już testy. Spowoduje to tak naprawdę powrót do samego początku. Czyli najpierw zmiana w interfejsach, pózniej poprawa szkieletu klasy, później poprawka testów, a dopiero później fix implementacji.

```ts
// definitions.ts
export const POSITIONS = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11] as const;
export const OCTAVES = [0, 1, 2, 3, 4, 5, 6, 7, 8] as const;
export const SHARP_NAMES = [
  "C",
  "C#",
  "D",
  "D#",
  "E",
  "F",
  "F#",
  "G",
  "G#",
  "A",
  "A#",
  "B",
] as const;
export const BMOLL_NAMES = [
  "C",
  "Db",
  "D",
  "Eb",
  "E",
  "F",
  "Gb",
  "G",
  "Ab",
  "A",
  "Bb",
  "B",
] as const;

export type NotePosition = typeof POSITIONS[number];
export type SharpNoteName = typeof SHARP_NAMES[number];
export type BmollNoteName = typeof BMOLL_NAMES[number];
export type NoteOctave = typeof OCTAVES[number];
export enum NoteNotationSymbol {
  Sharp = "#",
  Bmoll = "b",
}

export type NoteId = string | number;
export interface Note {
  position: NotePosition;
  octave: NoteOctave;
  id: NoteId;
  sharpName: string;
  bmollName: string;
}

// guitarNote.ts

import {
  Note,
  NotePosition,
  NoteOctave,
  NoteId,
  SHARP_NAMES,
  NoteNotationSymbol,
} from "./definitions";
import { getNextPosition } from "./utils";

export class GuitarNote implements Note {
  sharpName: string;
  bmollName: string;

  constructor(
    public position: NotePosition,
    public octave: NoteOctave,
    public id: NoteId
  ) {
    this.sharpName = SHARP_NAMES[position];
    this.bmollName = this._getBmollName(this.sharpName, position);
  }

  private _getBmollName = (name: string, position: NotePosition): string => {
    if (!this.sharpName.includes(NoteNotationSymbol.Sharp)) {
      return name;
    }

    return SHARP_NAMES[getNextPosition(position)] + NoteNotationSymbol.Bmoll;
  };
}
```

Teraz poprawa testów, które nawiązują do starych interfejsów. 

```ts
// guitarNote.test.ts
import { GuitarNote } from "./guitarNote";

describe("GuitarNote", () => {
  it("assigns given properties", () => {
    const note = new GuitarNote(0, 0, 0);
    expect(note.position).toBe(0);
    expect(note.octave).toBe(0);
    expect(note.id).toBe(0);
  });

  it("gets sharp name", () => {
    expect(new GuitarNote(0, 0, 0).sharpName).toBe("C");
    expect(new GuitarNote(1, 0, 0).sharpName).toBe("C#");
    expect(new GuitarNote(3, 0, 0).sharpName).toBe("D#");
    expect(new GuitarNote(6, 0, 0).sharpName).toBe("F#");
    expect(new GuitarNote(8, 0, 0).sharpName).toBe("G#");
    expect(new GuitarNote(10, 0, 0).sharpName).toBe("A#");
    expect(new GuitarNote(11, 0, 0).sharpName).toBe("B");
  });

  it("gets bmoll name", () => {
    expect(new GuitarNote(0, 0, 0).bmollName).toBe("C");
    expect(new GuitarNote(1, 0, 0).bmollName).toBe("Db");
    expect(new GuitarNote(3, 0, 0).bmollName).toBe("Eb");
    expect(new GuitarNote(6, 0, 0).bmollName).toBe("Gb");
    expect(new GuitarNote(8, 0, 0).bmollName).toBe("Ab");
    expect(new GuitarNote(10, 0, 0).bmollName).toBe("Bb");
    expect(new GuitarNote(11, 0, 0).bmollName).toBe("B");
  });
});
```

Znów zielono.

![image](https://user-images.githubusercontent.com/22937810/152338096-ba919b9d-254b-4744-bdff-4dc9b4602c6a.png)

Teraz została ostatnia rzecz. Chcemy pozbyć się **magic numbers**. W tym celu najpierw zmienimy API klasy **GuitarNote**, a dokładniej konstruktor. Teraz zamiast parametru **position**, przyjmie on parametr cos, który ma 3 możliwe warianty (**Position, SharpName, BmollName)**. Ułatwi to czytanie kodu.

Zmieniamy API:

```ts
// definitions
export type NoteSymbol = NotePosition | SharpNoteName | BmollNoteName; // dodajemy
// guitarNote.ts
constructor(
  public symbol: NoteSymbol, // wcześniej position: NotePosition
  public octave: NoteOctave,
  public id: NoteId
) {
  this.sharpName = SHARP_NAMES[position]; // i co teraz ? Tutaj jest error
  this.bmollName = this._getBmollName(this.sharpName, position); i co teraz ? Tutaj jest error
}
```

Teraz dochodzimy do sytuacji, w której nie pozostaje nam nic innego jak zakomentować kod, poprawić testy i napisać nową implementacje. Zmieniliśmy tak mocno kształt naszego API, że to jest najszybsza droga. Tutaj **refactor** już nie działa. Cofamy się do początku.

```ts
// guitarNote.ts
import {
  Note,
  NotePosition,
  NoteOctave,
  NoteId,
  SHARP_NAMES,
  NoteNotationSymbol,
} from "./definitions";
import { getNextPosition } from "./utils";

export class GuitarNote implements Note {
  sharpName = "";
  bmollName = "";
  position = 0 as NotePosition;

  constructor(
    public symbol: NoteSymbol,
    public octave: NoteOctave,
    public id: NoteId
  ) {
    // this.sharpName = SHARP_NAMES[position];
    // this.bmollName = this._getBmollName(this.sharpName, position);
  }

  private _getBmollName = (name: string, position: NotePosition): string => {
    if (!this.sharpName.includes(NoteNotationSymbol.Sharp)) {
      return name;
    }

    return SHARP_NAMES[getNextPosition(position)] + NoteNotationSymbol.Bmoll;
  };
}
```

Sprawdzanie typów przechodzi więc teraz testy, a później nowa implementacja.

```ts
// guitarNote.test.ts
import { GuitarNote } from "./guitarNote";

describe("GuitarNote", () => {
  it("assigns given properties", () => {
    const note = new GuitarNote(0, 0, 0);
    expect(note.position).toBe(0);
    expect(note.octave).toBe(0);
    expect(note.id).toBe(0);
  });

  describe("for position as symbol", () => {
    it("gives sharp name", () => {
      expect(new GuitarNote(0, 0, 0).sharpName).toBe("C");
      expect(new GuitarNote(1, 0, 0).sharpName).toBe("C#");
      expect(new GuitarNote(3, 0, 0).sharpName).toBe("D#");
      expect(new GuitarNote(6, 0, 0).sharpName).toBe("F#");
      expect(new GuitarNote(8, 0, 0).sharpName).toBe("G#");
      expect(new GuitarNote(10, 0, 0).sharpName).toBe("A#");
      expect(new GuitarNote(11, 0, 0).sharpName).toBe("B");
    });

    it("gives bmoll name", () => {
      expect(new GuitarNote(0, 0, 0).bmollName).toBe("C");
      expect(new GuitarNote(1, 0, 0).bmollName).toBe("Db");
      expect(new GuitarNote(3, 0, 0).bmollName).toBe("Eb");
      expect(new GuitarNote(6, 0, 0).bmollName).toBe("Gb");
      expect(new GuitarNote(8, 0, 0).bmollName).toBe("Ab");
      expect(new GuitarNote(10, 0, 0).bmollName).toBe("Bb");
      expect(new GuitarNote(11, 0, 0).bmollName).toBe("B");
    });
  });

  describe("for sharp name as symbol", () => {
    it("gives sharp name", () => {
      expect(new GuitarNote("C", 0, 0).sharpName).toBe("C");
      expect(new GuitarNote("C#", 0, 0).sharpName).toBe("C#");
      expect(new GuitarNote("D#", 0, 0).sharpName).toBe("D#");
      expect(new GuitarNote("F#", 0, 0).sharpName).toBe("F#");
      expect(new GuitarNote("G#", 0, 0).sharpName).toBe("G#");
      expect(new GuitarNote("A#", 0, 0).sharpName).toBe("A#");
      expect(new GuitarNote("B", 0, 0).sharpName).toBe("B");
    });

    it("gives bmoll name", () => {
      expect(new GuitarNote("C", 0, 0).bmollName).toBe("C");
      expect(new GuitarNote("C#", 0, 0).bmollName).toBe("Db");
      expect(new GuitarNote("D#", 0, 0).bmollName).toBe("Eb");
      expect(new GuitarNote("F#", 0, 0).bmollName).toBe("Gb");
      expect(new GuitarNote("G#", 0, 0).bmollName).toBe("Ab");
      expect(new GuitarNote("A#", 0, 0).bmollName).toBe("Bb");
      expect(new GuitarNote("B", 0, 0).bmollName).toBe("B");
    });
  });

  describe("for bmoll name as symbol", () => {
    it("gives sharp name", () => {
      expect(new GuitarNote("C", 0, 0).sharpName).toBe("C");
      expect(new GuitarNote("Db", 0, 0).sharpName).toBe("C#");
      expect(new GuitarNote("Eb", 0, 0).sharpName).toBe("D#");
      expect(new GuitarNote("Gb", 0, 0).sharpName).toBe("F#");
      expect(new GuitarNote("Ab", 0, 0).sharpName).toBe("G#");
      expect(new GuitarNote("Bb", 0, 0).sharpName).toBe("A#");
      expect(new GuitarNote("B", 0, 0).sharpName).toBe("B");
    });

    it("gives bmoll name", () => {
      expect(new GuitarNote("C", 0, 0).bmollName).toBe("C");
      expect(new GuitarNote("Db", 0, 0).bmollName).toBe("Db");
      expect(new GuitarNote("Eb", 0, 0).bmollName).toBe("Eb");
      expect(new GuitarNote("Gb", 0, 0).bmollName).toBe("Gb");
      expect(new GuitarNote("Ab", 0, 0).bmollName).toBe("Ab");
      expect(new GuitarNote("Bb", 0, 0).bmollName).toBe("Bb");
      expect(new GuitarNote("B", 0, 0).bmollName).toBe("B");
    });
  });
});
```

Mamy tutaj trochę duplikacji. Jednak, aby jej uniknąć musielibyśmy dodać jakąs logikę do testów. Testy mogłyby przestać działać nie ze względu na zła implementacje, ale zła logike testów. Tego powinniśmy unikać - to może generować **false positives/negatives**. W testach duplikacja nie jest takim problemem jak w przypadku kodu aplikacji. 
Możemy się jednak pokusić o małe usprawnienie tworzenia obiektu. Wykorzystamy wzorzec **factory function**.

```ts
// guitarNote.test.ts
import { NoteSymbol } from "./definitions";
import { GuitarNote } from "./guitarNote";

describe("GuitarNote", () => {
  const createGuitarNote = (symbol: NoteSymbol) => {
    return new GuitarNote(symbol, 0, 0);
  };

  it("assigns given properties", () => {
    const note = createGuitarNote(0);
    expect(note.position).toBe(0);
    expect(note.octave).toBe(0);
    expect(note.id).toBe(0);
  });

  describe("for position as symbol", () => {
    it("gives sharp name", () => {
      expect(createGuitarNote(0).sharpName).toBe("C");
      expect(createGuitarNote(1).sharpName).toBe("C#");
      expect(createGuitarNote(3).sharpName).toBe("D#");
      expect(createGuitarNote(6).sharpName).toBe("F#");
      expect(createGuitarNote(8).sharpName).toBe("G#");
      expect(createGuitarNote(10).sharpName).toBe("A#");
      expect(createGuitarNote(11).sharpName).toBe("B");
    });

    it("gives bmoll name", () => {
      expect(createGuitarNote(0).bmollName).toBe("C");
      expect(createGuitarNote(1).bmollName).toBe("Db");
      expect(createGuitarNote(3).bmollName).toBe("Eb");
      expect(createGuitarNote(6).bmollName).toBe("Gb");
      expect(createGuitarNote(8).bmollName).toBe("Ab");
      expect(createGuitarNote(10).bmollName).toBe("Bb");
      expect(createGuitarNote(11).bmollName).toBe("B");
    });
  });

  describe("for sharp name as symbol", () => {
    it("gives sharp name", () => {
      expect(createGuitarNote("C").sharpName).toBe("C");
      expect(createGuitarNote("C#").sharpName).toBe("C#");
      expect(createGuitarNote("D#").sharpName).toBe("D#");
      expect(createGuitarNote("F#").sharpName).toBe("F#");
      expect(createGuitarNote("G#").sharpName).toBe("G#");
      expect(createGuitarNote("A#").sharpName).toBe("A#");
      expect(createGuitarNote("B").sharpName).toBe("B");
    });

    it("gives bmoll name", () => {
      expect(createGuitarNote("C").bmollName).toBe("C");
      expect(createGuitarNote("C#").bmollName).toBe("Db");
      expect(createGuitarNote("D#").bmollName).toBe("Eb");
      expect(createGuitarNote("F#").bmollName).toBe("Gb");
      expect(createGuitarNote("G#").bmollName).toBe("Ab");
      expect(createGuitarNote("A#").bmollName).toBe("Bb");
      expect(createGuitarNote("B").bmollName).toBe("B");
    });
  });

  describe("for bmoll name as symbol", () => {
    it("gives sharp name", () => {
      expect(createGuitarNote("C").sharpName).toBe("C");
      expect(createGuitarNote("Db").sharpName).toBe("C#");
      expect(createGuitarNote("Eb").sharpName).toBe("D#");
      expect(createGuitarNote("Gb").sharpName).toBe("F#");
      expect(createGuitarNote("Ab").sharpName).toBe("G#");
      expect(createGuitarNote("Bb").sharpName).toBe("A#");
      expect(createGuitarNote("B").sharpName).toBe("B");
    });

    it("gives bmoll name", () => {
      expect(createGuitarNote("C").bmollName).toBe("C");
      expect(createGuitarNote("Db").bmollName).toBe("Db");
      expect(createGuitarNote("Eb").bmollName).toBe("Eb");
      expect(createGuitarNote("Gb").bmollName).toBe("Gb");
      expect(createGuitarNote("Ab").bmollName).toBe("Ab");
      expect(createGuitarNote("Bb").bmollName).toBe("Bb");
      expect(createGuitarNote("B").bmollName).toBe("B");
    });
  });
});
```

Teraz fix implementacji i koniec. 


