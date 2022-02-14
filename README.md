# Spigot/Paper Exploit Fixes

This document offers configuration suggestions for modifying/disabling changes made
by Spigot and Paper which may interfere with unintentional gameplay emergence.

See [explanation](#explanation) at the bottom of the document for more information.

## Common Issues
- [Duping](#duping)
  - [Sand/gravity block duping isn't working](#sand-dupe)
  - [TNT duping isn't working](#tnt-duping-isnt-working)
  - [TNT in world eater isn't falling](#tnt-in-world-eater-isnt-falling)
- [Items](#items)
  - [Items despawned in the nether after dying](#items-despawned-in-the-nether-after-dying)
  - [Some shulker loaders not picking up items](#hopper-cooldown)
- [Spawning](#spawning)
  - [Mob switch isn't working](#mob-switch-isnt-working)
- [Villagers](#villagers)
  - [Prices increase substantially after one trade](#trade-demand) 
  - [Zombie villager cures only discount once](#curing-discount)
- [Misc.](#misc)
  - [Bedrock isn't breaking](#bedrock)
  - [Ender-porter doesn't work](#ender-porter)
  - [The nether and the end stay loaded](#keep-spawn-loaded)
  - [Unlimiting Packets](#unlimiting-packets)

### Duping

<a name="sand-dupe"/>

#### Sand/gravity block duping isn't working

Gravity block duplication is patched by Paper with no configuration offered to re-enable.
You can see the discussion behind this at
[PaperMC/Paper#3724](https://github.com/PaperMC/Paper/issues/3724).

You can replace Paper with a fork which undoes, or modifies, the patch. A popular Paper
fork that does this is [Purpur](https://purpurmc.org).

In `purpur.yml` set the following:
```yaml
blocks:
  sand:
    fix-duping: false
```

#### TNT duping isn't working

Duplication via flawed piston logic is patched by Paper, but can be re-enabled.

In `paper.yml` set the following:
```yaml
unsupported-settings:
  allow-piston-duplication: true
```

_Please be aware that this will also enable other forms of piston-caused duping, 
such as carpet duping and rail duping._

#### TNT in world eater isn't falling

Spigot limits the amount of TNT entities that can be processed per game tick. World eaters use
a very large amount of TNT entities to eat the world and will very easily hit this limit
where its physics and aging won't update. This is configurable.

In `spigot.yml` change the following setting:
```yaml
max-tnt-per-tick: 1000
```

Try and calculate the amount of TNT active at a given time during your world eating and set a max
value slightly above this, or alternatively, you can set this absurdly high.

_Unfortunately Spigot does not offer a value that outright disables this._

### Items

#### Items despawned in the nether after dying

The chunks the items were in were loaded and the 5 minute timer expired. If no players were around
to load the chunks, it could be that the items were in the spawn chunks. In CraftBukkit,
even the nether and the end have spawn chunks which are always loaded.

See [The nether and the end stay loaded](#keep-spawn-loaded) for more information.

<a name="hopper-cooldown"/>

#### Some shulker loaders not picking up items

Paper makes optimizations to the logic of hoppers, however, included is a change in how
hoppers cooldown. In vanilla, a hopper only cools-down when an item is moved to or from
its inventory. In Paper, a hopper cools-down when an item movement is attempted between
inventories. 

This shouldn't be an issue when using a hopper chain as they only move items at a rate
of 8 game ticks, the same duration as the cooldown, nor should it be an issue with any
single-speed item filter because the hopper points into a non-inventory block entity,
so no item push is attempted.

In `paper.yml` set the following:
```yaml
hopper:
  cooldown-when-full: false
```

### Spawning

#### Mob switch isn't working

By default, Paper only counts natural spawns towards the mobcap. Your mobswitch might not
be working because the mobs in it are not natural spawns.

In `paper.yml` set the following:
```yaml
count-all-mobs-for-spawning: true
```

_Please be aware that CraftBukkit messes with the persistence code of mobs and can cause mobs
to be counted who otherwise wouldn't be in Vanilla. By enabling this config option, this issue
may occur and you might experience a slow down in mob farms. But you're enabling this because
you want to disable mob spawns with your mob switch, right?_

### Villagers

<a name="trade-demand"/>

#### Prices increase substantially after one trade
When villagers restock, demand is updated for all trade offers. Demand decreases the less a
player purchases a particular offer, however, this decrease has no lower bounds and will
continuously decrease with every restock towards negative infinity. This issue is tracked by
[MC-163962](https://bugs.mojang.com/browse/MC-163962).

In vanilla, the longer you don't trade, the further in to the negatives demand goes, and the
more you need to trade before demand goes back in to the positives and prices increase. The
dramatic price increase does occur in vanilla but it takes significantly longer for it to 
occur.

Paper patches this in [0436-Fix-villager-trading-demand-MC-163962.patch](https://github.com/PaperMC/Paper/blob/a8f2d67/patches/server/0436-Fix-villager-trading-demand-MC-163962.patch)
where a lower bound of zero is enforced. No configuration exists in Paper to disable this fix
or configure this lower bound.

Purpur, a fork of Paper, offers a configuration option for this lower bound. In `purpur.yml`
change the following:
```yaml
mobs:
  villager:
    minimum-demand: 0
```

Configure this to a value below zero. A good starting value might be `-100`. This should allow
the player to trade for around 4-5 in-game days before the price starts increasing. Adjust
lower if necessary.

<a name="curing-discount"/>

#### Zombie villager cures only discount once
Curing zombie villagers and stacking discounts is considered a bug, as tracked by
[MC-181190](https://bugs.mojang.com/browse/MC-181190).

Mojang has marked this as important, implying there is an issue in some form. How much is unclear.
Maybe stacking is intended but not as much as it does. Maybe they intend to rework the price
algorithm so the discount isn't so extreme. It's unknown due to the lack of feedback from Mojang.

Regardless, this is patched in Paper. When you cure a zombie villager, all previous `MAJOR_POSITIVE`
reputation is removed for the player preventing you from stacking up the reputation to decrease
prices further. An option is offered to disable this fix.

In `paper.yml` set the following:
```yaml
game-mechanics:
  fix-curing-zombie-villager-discount-exploit: false
```

### Misc.

<a name="bedrock"/>

#### Bedrock isn't breaking

Bedrock is, by design, not intended to be broken. Paper patches bedrock breaking 
but offers a config option to re-enable this exploit.

In `paper.yml` set the following:
```yaml
unsupported-settings:
  allow-permanent-block-break-exploits: true
  allow-headless-pistons: true
```

`allow-permanent-block-break-exploits` allows the bedrock block to be broken  
`allow-headless-pistons` enables the common exploit used to break bedrock

If you're still having difficulties with breaking bedrock, see [Unlimiting Packets](#unlimiting-packets).

<a name="ender-porter"/>

#### Ender-porter doesn't work

Long distance traveling via ender pearl stasis is tracked by 
[MC-129119](https://bugs.mojang.com/browse/MC-129119) which remains unresolved.

Paper offers a configuration option to disable this mechanic by removing the thrower's
UUID when the entity is loaded from file (i.e. chunk loading). When the pearl breaks,
the thrower is unknown and therefore the player is not teleported.

This option exists for server owners who wish to stop their players from exploiting
this, and is enabled by default.

In `paper.yml` set the following:
```yaml
game-mechanics:
  disable-unloaded-chunk-enderpearl-exploit: false
```

<a name="keep-spawn-loaded" />

#### The nether and the end stay loaded

In vanilla, an area around the world spawn point (i.e. spawn chunks) always stays loaded,
but this is only in the overworld. In CraftBukkit, the nether and the end are split
from the overworld into 2 additional, separate worlds, presumably designed as such for
multi-world or multi-dimensional support. The spawn chunks of each world always stay
loaded, including the nether and the end.

Mostly a non-issue, but if a player dies in the nether or the end and takes longer than
5 minutes to return to their place of death they may find nothing but their XP remaining.
This is because they died in that world's spawn chunks where their items stayed loaded,
allowing them to age and eventually expire without the presence of a player. This doesn't
happen with XP because XP requires a player to be close by to "activate" and age it.
None of this occurs in vanilla gameplay, and such a change may lead to frustration for
players who happen to experience loss from it.

To disable spawn chunks in the nether and the end, make the following change in `paper.yml`:
```yaml
world-settings:
  default:
    keep-spawn-loaded: true
  <WORLDNAME>_nether:
    keep-spawn-loaded: false
  <WORLDNAME>_the_end:
    keep-spawn-loaded: false
```
Spawn is loaded by default for all worlds, however we make exceptions for the nether
and the end by making world-specific settings for them. You must make sure the world name
matches the name of your world. For example, if your world is called `world` (the default name),
you will need to create world-specific settings for `world_nether` and `world_the_end`.

<a name="unlimiting-packets" />

#### Unlimiting Packets

Paper comes with a built-in configuration for limiting incoming packets on the server. This
can cause problems for using autoclickers or spam-click mods when breaking bedrock, or when
using mods to autocraft (eg: Itemscroller) which make use of the extra packets to guarantee 
certain actions. To change this behaviour simply modify the following config (`paper.yml`):

```yaml
settings:
  packet-limiter:
    limits:
      all:
# Default values:
        interval: 7.0
        max-packet-rate: 500.0
# Default value:
  incoming-packet-spam-threshold: 300
```

If you would like to preserve some of the spam limiting without disabling them (to do that,
use `0` for the configuration values) we have found that autoplace mods/bedrock breaking do
work consistently with `incoming-packet-spam-threshold` set to `30` which is the value used
by Spigot.

## Explanation

CraftBukkit modifies the vanilla server JAR to implement the Bukkit API, and API used
by plugins to hook in to, and modify, the behavior of Minecraft.

Spigot is a fork of CraftBukkit which allows certain constants of the game be configurable,
such as mob caps, item merge radius, entity tracking range, and more. It also introduces
many features, such as the entity activation range, to reduce the amount of processing
the server has to do every game tick.

Paper is a fork of Spigot which fixes some issues created by CraftBukkit and Spigot,
fixes exploits, and adds more configurable gameplay aspects. Paper also adds optimisation
patches which improves the performance of the server. Paper is the most widely used server
in Minecraft and is [recommended over Spigot](https://madelinemiller.dev/blog/paper-vs-spigot/).

All of these culminate in an altered experience, which may not be noticeable to most survival
players, but for those wishing to venture out to the more technical side of Minecraft may
encounter issues when playing with this server software.

If you want to adopt a more technical gameplay style, I recommend you try using a Fabric server
with optimization mods such as [Lithium](https://github.com/CaffeineMC/lithium-fabric) and
[Starlight](https://github.com/PaperMC/Starlight) (or 
[Phosphor](https://github.com/CaffeineMC/phosphor-fabric) if you're interested in exploring
light suppression).

If Fabric isn't an option for you, or you need to run Bukkit plugins, then there are configuration
options available for you to change to restore the more emergent style of gameplay performed
by the Technical Minecraft community. Find your issue [above](#common-issues) and make the
appropriate changes.
