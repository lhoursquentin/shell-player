# shell-player

Experimental shell based sound player controlled via a fifo.

## What this is not doing
This does not handle the actual decoding and playback, this can be done via
vlc, mpg123 or any other player that can exit after playing a sound.

## What this is doing
The goal is to provide a very basic API over those players to control pause,
next, previous, sound repeat, quit and fetch playback information.

This allows to script both control and playback information retrieval in a
simple and unified manner.

# Architecture (with web controller example)

```
                                  Pending state?
                             Re-enqueue info request
                            +-----------------------+
                            |                       |
             shell-player   |                       |                                Web controller
                            |                       |
        +--------------------------+                |                            +-------------------------+
        |                   |      |                |                            |                         |
        |                   |      |                v          next/loop/quit    |    +---------------+    |
        |    +--------------+-+    |        +-------+-------+  pause/previous    |    |               |    |
        |    |                |    |        |               |                    |    |   Request     |    |
 +-----------+ command reader +<------------+ commands fifo +<------------------------+   processor   |    |
 |      |    |                |    |        |               |                    |    |               |    |
 |      |    +--------+-------+    |        +-------+-------+<---------+         |    +--+----------+-+    |
 |      |        Kill | Start      |                ^                  |         |       ^          |      |
 |      |             v            |                |                  |         |       |          v      |
 |      |    +--------+-------+    |                |                  |  info   |       |      +---+----+ |
 |      |    |                |    |    Player      |                  +------------------------+ fetch  | |
 |      |    |  song player   |    |     end        |                            |       |      | info   | |
 |      |    | (cvlc/mpg123)  +---------------------+                  +----------------------->+        | |
 |      |    |                |    |                                   |         |       |      +---+----+ |
 |      |    +----------------+    |                                   |         |       |          |      |
 |      |                          |                                   |         |       |          v      |
 |      |    +----------------+    |        +---------------+          |         |       |      +---+----+ |
 |      |    |                |    |        |               |          |         |       |      |        | |
 +---------->+  Info caster   +------------>+   Info fifo   +----------+         |       |      |  HTML  | |
 Info   |    |                |    |        |               |                    |       |      |        | |
request |    +----------------+    |        +---------------+                    |       |      +---+----+ |
        |                          |                                             |       |          |      |
        +------------+-------------+                                             |       |          |      |
                     ^                                                           |       |          |      |
                     |                                                           |       |          v      |
                     |                                                           |   +---+--+    +--+---+  |
                     +--------------------+------------------------------------->+   |      |    |      |  |
                                          |                                      |   |  rx  |    |  tx  |  |
                                          |                                      |   | fifo |    | fifo |  |
                                          |                                      |   |      |    |      |  |
                                   Launch |                                      |   +----+-+    +-+----+  |
                                          |                                      |        ^        |       |
                                          |                                      |        |        v       |
                                     +----+-----+  Interact  +-------------+     |     +--+--------+--+    |
                                     |          |            |             |     |     |              |    |
                                     |   User   +----------->+   Browser   +<--------->+    netcat    |    |
                                     |          |            |             |     |     |              |    |
                                     +----------+            +-------------+     |     +--------------+    |
                                                                                 |                         |
                                                                                 +-------------------------+
```
