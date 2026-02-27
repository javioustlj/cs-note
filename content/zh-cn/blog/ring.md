

```

node 0

// intra
0 --> 16 --> 17 --> 1 --> 2 --> 18 --> 19 --> 3 --> 6 --> 22 --> 23 --> 7 --> 4 --> 20 --> 21 --> 5
// Preset
prev: -1
next: 16
send: 5
recv: 0
// Postset
prev: -1
next: 16
send: 5 5 5 5 5 5 5 5 13 13 13 13 13 13 13 13
recv: 0 0 0 0 0 0 0 0 8 8 8 8 8 8 8 8
// connectRing
firstRanks: 0 8

node 1

// intra
8 --> 24 --> 25 --> 9 --> 10 --> 26 --> 27 --> 11 --> 14 --> 30 --> 31 --> 15 --> 12 --> 28 --> 29 --> 13
//Preset
prev: -1
next: 24
send: 13
recv: 8
//Postset
send: 5 5 5 5 5 5 5 5 13 13 13 13 13 13 13 13
recv: 0 0 0 0 0 0 0 0 8 8 8 8 8 8 8 8
// connectRing
firstRanks: 0 8

```



```

node 0

// intra
0 --> 16 --> 17 --> 1 --> 2 --> 18 --> 19 --> 3 --> 6 --> 22 --> 23 --> 7 --> 4 --> 20 --> 21 --> 5
// Preset
prev: -1
next: 16
send: 5
recv: 0
send1: 1
recv1: 2
// Postset
prev: -1
next: 16
send: 5 5 5 5 5 5 5 5 13 13 13 13 13 13 13 13
recv: 0 0 0 0 0 0 0 0 8 8 8 8 8 8 8 8
send1: 1 1 1 1 1 1 1 1 9 9 9 9 9 9 9 9
recv1: 2 2 2 2 2 2 2 2 10 10 10 10 10 10 10 10
// connectRing
firstRanks: 0 8

node 1

// intra
8 --> 24 --> 25 --> 9 --> 10 --> 26 --> 27 --> 11 --> 14 --> 30 --> 31 --> 15 --> 12 --> 28 --> 29 --> 13
//Preset
prev: -1
next: 24
send: 13
recv: 8
send1: 9
recv1: 10
//Postset
send: 5 5 5 5 5 5 5 5 13 13 13 13 13 13 13 13
recv: 0 0 0 0 0 0 0 0 8 8 8 8 8 8 8 8
send1: 
recv1: 
// connectRing
firstRanks: 0 8

```
