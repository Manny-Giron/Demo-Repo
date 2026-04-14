# CSC411 Bomb 31

## Readme Portion

- Name: `Emmanuel Giron`
- Bomb number: `31`
- Collaboration/help: `[Acknowledge help, collaboration, office hours, AI use, etc.]`
- Approximate hours spent: `8`

## Phases Portion

### Input Lines

```text
I am just a renegade hockey mom.
1 2 4 8 16 32
0 506
10 5
mfcdhg
5 4 3 6 2 1
```

### Informal Explanations

#### Phase 1

Phase 1 reads a line and compares it against one fixed string. The bomb explodes unless the input is exactly:

```text
I am just a renegade hockey mom.
```

#### Phase 2

Phase 2 reads six integers. The first integer must be `1`, and every later integer must be exactly twice the previous integer. So the sequence must be a geometric progression with ratio `2`, starting at `1`.

#### Phase 3

Phase 3 reads two integers: an index and a value. The first integer must be between `0` and `7`, and the program uses it as a switch/jump-table index to choose one constant. The second integer must match the constant selected by the first integer.

One valid input is:

```text
0 506
```

The full mapping is:

- `0 -> 506`
- `1 -> 411`
- `2 -> 239`
- `3 -> 488`
- `4 -> 321`
- `5 -> 145`
- `6 -> 163`
- `7 -> 830`

#### Phase 4

Phase 4 reads two integers. The first must be at most `14`. It then calls `func4(first, 0, 14)`, where `func4` performs a recursive binary-search-like walk and returns a number that encodes the path taken. The bomb explodes unless the return value is `5`, and the second input must also be exactly `5`.

The accepted input is:

```text
10 5
```

#### Phase 5

Phase 5 reads a 6-character string. For each character, it keeps only the low 4 bits and uses that value as an index into the table:

```text
maduiersnfotvbyl
```

The 6 mapped characters must spell:

```text
bruins
```

The input `mfcdhg` works because the low nibbles of its ASCII bytes are `d 6 3 4 8 7`, which map to `b r u i n s`.

#### Phase 6

Phase 6 reads six distinct integers from `1` to `6`. It checks that all six are in range and that there are no duplicates. Then it transforms each number `x` into `7 - x`, uses those transformed numbers as positions in a linked list, rebuilds the list in that order, and finally checks that the node values are in nonincreasing order.

The original node values are:

- node 1: `640`
- node 2: `970`
- node 3: `744`
- node 4: `648`
- node 5: `530`
- node 6: `266`

So the descending order of node positions is `2, 3, 4, 1, 5, 6`. Because the code transforms each input with `7 - x`, the required user input is:

```text
5 4 3 6 2 1
```

## Code Section

### Phase 1

```c
void phase_1(char *input) {
    if (strings_not_equal(input, "I am just a renegade hockey mom.") != 0) {
        explode_bomb();
    }
}
```

### Phase 2

```c
void phase_2(char *input) {
    int a[6];
    read_six_numbers(input, a);

    if (a[0] != 1) {
        explode_bomb();
    }

    for (int i = 0; i < 5; i++) {
        if (a[i + 1] != 2 * a[i]) {
            explode_bomb();
        }
    }
}
```

### Phase 3

```c
void phase_3(char *input) {
    int index;
    int value;
    int expected;

    if (sscanf(input, "%d %d", &index, &value) <= 1) {
        explode_bomb();
    }

    switch (index) {
    case 0: expected = 506; break;
    case 1: expected = 411; break;
    case 2: expected = 239; break;
    case 3: expected = 488; break;
    case 4: expected = 321; break;
    case 5: expected = 145; break;
    case 6: expected = 163; break;
    case 7: expected = 830; break;
    default:
        explode_bomb();
        expected = 0;
        break;
    }

    if (value != expected) {
        explode_bomb();
    }
}
```

### `func4`

```c
int func4(int target, int low, int high) {
    int mid = low + (high - low) / 2;

    if (target < mid) {
        return 2 * func4(target, low, mid - 1);
    }

    if (target > mid) {
        return 2 * func4(target, mid + 1, high) + 1;
    }

    return 0;
}
```

### Phase 4

```c
void phase_4(char *input) {
    int x;
    int y;

    if (sscanf(input, "%d %d", &x, &y) != 2) {
        explode_bomb();
    }

    if (x > 14) {
        explode_bomb();
    }

    if (func4(x, 0, 14) != 5) {
        explode_bomb();
    }

    if (y != 5) {
        explode_bomb();
    }
}
```

### Phase 5

```c
void phase_5(char *input) {
    char out[7];
    char *table = "maduiersnfotvbyl";

    if (string_length(input) != 6) {
        explode_bomb();
    }

    for (int i = 0; i < 6; i++) {
        out[i] = table[input[i] & 0xF];
    }

    out[6] = '\0';

    if (strings_not_equal(out, "bruins") != 0) {
        explode_bomb();
    }
}
```

### Phase 6

```c
typedef struct Node {
    int value;
    int index;
    struct Node *next;
} Node;

void phase_6(char *input) {
    int a[6];
    Node *nodes[6];
    Node *p = NULL;

    read_six_numbers(input, a);

    for (int i = 0; i < 6; i++) {
        if (a[i] < 1 || a[i] > 6) {
            explode_bomb();
        }

        for (int j = i + 1; j < 6; j++) {
            if (a[i] == a[j]) {
                explode_bomb();
            }
        }
    }

    for (int i = 0; i < 6; i++) {
        a[i] = 7 - a[i];
    }

    for (int i = 0; i < 6; i++) {
        int position = a[i];
        p = &node1;

        for (int step = 1; step < position; step++) {
            p = p->next;
        }

        nodes[i] = p;
    }

    for (int i = 0; i < 5; i++) {
        nodes[i]->next = nodes[i + 1];
    }
    nodes[5]->next = NULL;

    for (int i = 0; i < 5; i++) {
        if (nodes[i]->value < nodes[i + 1]->value) {
            explode_bomb();
        }
    }
}
``
