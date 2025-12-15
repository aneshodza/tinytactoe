# TinyTacToe
Have you ever wondered how small the needed storage for a game of tictactoe (including its history) is? This project tries to tackle that question
## Introduction
This project is a proof-of-concept on how small a game of tictactoe (including move history) can be encoded without any extra overhead like a Huffman-Tree.  
This project brings a game down to **23 bits**, but the storage is done in an u32 (unsigned 32bit integer), as machines cannot assign single bits.

## Trying the project
### 1. Getting the project
If you want to get the project onto your own machine, you have two options: Clone and build it yourself or download it for your machine.
#### Downloading the project
If you just want to download the project, please do that here for your machine:
* [Linux](https://drive.google.com/uc?export=download&id=1iX4Iq5KXK38x2_sv_qPkL7wAtGUq8nS8)
* [Windows (x86)](https://drive.google.com/uc?export=download&id=1W0BpVFOqLISQAX_Z3FP3guZf-doIEUuV)
* [MacOS (ARM)](https://drive.google.com/uc?export=download&id=10_o0qdmZh3KS1GVpC3deCeKLTo8-i92c)

**Keep in mind:** These were only tested for certain versions of the operating systems listed. There is a chance it
won't work for you.

#### Building the project
If you want to build it yourself, you first need to pull it:
```
git clone git@github.com:aneshodza/minimal-tictactoe.git
cd ../minimal-tictactoe
```
Then you build the project using `cargo`:
```
cargo build --release
cd target/release/
```

### 2. Running the project
To then run the project, you move into the folder you downloaded it into and run it using the following command:
```
./minimal-tictactoe
```
#### Permission issues
There is a possibility that it won't let you run the program, as its permissions are not correct. To fix that run:
```
chmod +x minimal-tictactoe
```
If you retry running the code, it should work.

### 3. Using the project
**Important:** Currently, the project doesn't support really playing the game, but rather just inputting the moves or game
that needs to be encoded or decoded.
After running the project, you will be greeted with the following screen:
```
Tictactoe encoded in only 23 bits
What do you want to do?
  1. Decode your game
  2. Encode a game
  3. Exit
```
Here you can choose between either encoding or decoding the game. For the instructions of each, please refer to the
following sections.
#### Encoding
When you want to encode a game, the following prompt will appear:
```
Tictactoe encoded in only 23 bits
What do you want to do?
  1. Decode your game
  2. Encode a game
  3. Exit
2
Please enter 1 to 9 numbers (1-9) separated by spaces:
```
There you need to give the program all the moves you made separated by spaces. The game from the explanation below
would look like this:
```
Please enter 1 to 9 numbers (1-9) separated by spaces:
1 4 5 3 2 3 2 1 1
```
Then the program will output the encoded game:
```
Your encoded number is: 4310116
It's binary representation: 10000011100010001100100 (23 bit)

X O X
O O X
X O X
```
#### Decoding
When you want to decode a game, the following prompt will appear:
```
Tictactoe encoded in only 23 bits
What do you want to do?
  1. Decode your game
  2. Encode a game
  3. Exit
1
Please enter your game (in decimal)
```
There you need to give the program the encoded game. If we want to decode the game from the example above, we would
give it the following input:
```
Please enter your game (in decimal)
4310116
```
Then the program will output the decoded game:
```
X O X
O O X
X O X
```

## Encoding the game
### Getting the moves
The game numbers all squares on the ticktacktoe board from one to nine, as follows:
```
 1 | 2 | 3
-----------
 4 | 5 | 6
-----------
 7 | 8 | 9
```
When a user then plays a move, like on field 3, that index gets popped off the list, and the list gets
re-numbered:
```
 1 | 2 | X
-----------
 3 | 4 | 5
-----------
 6 | 7 | 8
```
And so-on and so forth for every of the 9 moves.

In the end, there should be an Array of nine numbers:
```
[1, 4, 5, 3, 2, 3, 2, 1, 1]
```
Every index of the array gets subtracted by one, as the binary representation can be shorter that way:
```
[0, 3, 4, 2, 1, 2, 1, 0, 0]
```
### Parsing them into a 23-bit number
#### Length of the moves
Every number occupies as little bits as are needed to represent all options.
As the board starts with nine options (zero to eight), the first number needs four bit.
The second then only needs three, as there are only eight options left (zero to seven), etc.
The constant `BIT_SIZES` is used to represent that:
```rust
pub const BIT_SIZES: [u8; 9] = [4, 3, 3, 3, 3, 2, 2, 1, 1];
```
The reason the last move cannot be left out (as there is only one option left) is that the game can end before the last
move is played.
Giving it one bit is far more efficient than any other option (like showing the running length).
#### Encoding the game
The concept is that there is a `shift` that pinpoints where in the number we currently are.
It can be imagined as follows:

![docs/shift_init.png](docs/shift_init.png)
The encoding starts by finding out how long our number will be and storing it into the `shift`:
```rust
let mut shift: u8 = BIT_SIZES[..moves.len()].iter().sum();
```
This looks as follows (if our game is all nine moves long):

![docs/shift_sum.png](docs/shift_sum.png)
Then we take a `SIGNIFIER`, which is just a one and shift it by the `shift`:
```rust
output |= (SIGNIFIER as u32) << shift;
```
The purpose of the signifier is to make a certain number of moves
always have the same number of bits used, as a game where every move is on the first field would lose information
to zero-padding.
We also have to subtract the shift by the size of the first move, to prepare space for the next move:
```rust
shift -= BIT_SIZES[0];
```
This looks as follows:
![docs/first_shift.png](docs/first_shift.png)
Then we loop through every move in the `moves` array and follow the same procedure:
1. Take the current move and subtract one from it
2. Shift it by the `shift`
3. Add it to the `output`, using a bitwise OR
4. Subtract the `shift` by the size of the next move
5. Repeat until all moves are encoded

Implemented in rust, it looks as follows:
```rust
for (idx, element) in moves.iter().enumerate() {
    output |= (((element - 1) as u32) << shift) as u32;
    if shift > 0 {
        shift -= BIT_SIZES[idx + 1];
    }
}
```
And visually, every move does about this:

![docs/loop.png](docs/loop.png)

## Decoding the game
Coming soon...

*Made with <3 by Anes Hodza*
