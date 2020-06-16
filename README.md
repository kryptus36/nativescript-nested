Apologies for the wall of text...

I am having a hard time understanding the data flow for my nested objects. The below code successfully updates my underlying data, but the view doesn't update. I think it's because my objects aren't observable all the way down the tree. Hopefully there's enough info here to make my question clear. My ultimate question is: how do I make sure the view updates when my nested data changes?

Secondary question: Would this be easier in vue / angular? I want to change to vue, but wanted to clean up my existing code (which worked, but was a horrible mess of classes).

Detail:

I have bingo-page-vm.ts, which has:
```typescript
const cards = ObservableArray<BingoCard>;
```

This array is passed in to the nativescript pager plugin:
```xml
<pager:Pager items="{{ cards }}" selectedIndex="{{ cardIndex }}">
  <pager:Pager.itemTemplate>
    <FlexboxLayout flexDirection="column" alignItems="center">
      <bingocard:bingocard />
    </FlexboxLayout>
  </pager:Pager.itemTemplate>
</pager:Pager> 
```

Bingo cards look like:

```typescript
export interface BPCardSpace {
  number: number;
  daubed: boolean;
}

interface BPBingoCardData {
  //each array has 5 BPCardSpace
  b: Array<BPCardSpace>;
  i: Array<BPCardSpace>;
  n: Array<BPCardSpace>;
  g: Array<BPCardSpace>;
  o: Array<BPCardSpace>;
}

export interface BPBingoCard {
  id: string;
  win: boolean;
  loading: boolean;

  data: BPBingoCardData;
}
```

Now, obviously my bingocard is held in a component. I then have a function to toggle a spaces daubed property:

```xml
<Repeater row="1" col="0" items="{{ data.b }}">
  <Repeater.itemTemplate>
    <Label text="{{ $value.number }}" loaded="centerLabel" tap="toggleDaub" />
  </Repeater.itemTemplate>
</Repeater>
```

In bingocard.ts:

```typescript
export function toggleDaub(args: EventData) {
  const label = <Label>args.object;

  if(label.bindingContext.number === 0) {
    //free space...ignore it
    return;
  }

  container.page.bindingContext.toggleDaub(label.bindingContext.number);
}
```

Then in my bingo-page-vm.ts:

```typescript
toggleDaub(n: number) {
  const card = this.cards.getItem(this.cardIndex);

  const cols = ['b', 'i', 'n', 'g', 'o'];
  const colIndex = cols.findIndex((col) => {
    return card.data[col].some((row) => {
      return row.number === n;
    });
  });

  const rowIndex = card.data[cols[colIndex]].findIndex((row) => {
    return row.number === n;
  });

  const item = card.data[cols[colIndex]][rowIndex];

  item.daubed = !item.daubed;
  card.data[cols[colIndex]][rowIndex] = item;
  this.cards.setItem(this.cardIndex, card);
}
```
