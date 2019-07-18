# Order Expansion
Order expansion refers to the automatic creation of suborders (secondary order) that are needed to fulfill your primary orders (what you're asking Lisbeth to make). The reason why I call it "expansion" is because the algorithm that creates suborders does so by expanding a graph of "nodes", each node being a primary or secondary order. When doing this, your inventory and that of your retainers is taken into account, and gets "consumed" first before creating suborders when needed.

When your inventory items are not enough to fulfill the requirements of a primary order, the algorithm looks for "item sources" that provide the required item. These sources are filtered based on your character's capabilities. For example, if a primary order needs Iron Ingots as an ingredient and your character doesn't have a gearset for Blacksmith (or the job is too low level), it'll use Armorer instead. Iron Ingots can also be purchased from vendors, so that also counts as an "item source". In the end, it chooses the source based on your setting's priorities (General Settings tab). This is why it's important to tell Lisbeth what your character can and cannot do, through Gearsets, Fly Unlocks, Book Unlocks, Reputations, etc.

The algorithm then calculates amounts for each suborder (always consuming your inventory first), and further expands these suborders until all orders are fulfilled. It then consolidates similar based on several parameters. That way you don't end up with two suborders for Iron Ingots if two of your primary orders ask for them, but rather a single combined suborder is created. An important thing to note about this is that suborders are only consolidated when they are "compatible". For example, if one suborder is made as ForceHQ and another as QuickSynth, they are not compatible, and are kept as separate suborders (since their execution is different).

Gathering orders are very particular in that they don't care where the item is gathered from when expanding, only that your character is capable of gathering it somehow. This is because Lisbeth tries to be as efficient as possible in regards to gathering things, and so which "item source" is good depends on the real time circumstances of your character once its running. For example, let's say you make two orders, one for Earth Crystals and another for Lignum Vitae Logs. The Earth Crystals can be found on a lot of nodes all over, but they are also found on the same node as the logs. So it doesn't choose where to gather them until execution, depending on several factors like proximity. In general it will try to gather from nodes that provide the most order items that don't have those items as hidden, and that are in the same area as your character at that moment. In our example, it'll go to the Lignum Vitae Log node and gather the Earth Crystals from there, since the node provides for both orders. This is why it's difficult to predict which node it will use if there's multiple options available, and can vary from character to character depending on their capabilities.

## Order Types
There are different kinds of orders, which are categorized as follows:

**Production Orders**

1. Craft
2. Gather
3. Grind
4. Purchase
5. Exchange
6. Craft Masterpiece
7. Gather Masterpiece

These orders are considered "production" orders, which means they can be created as suborders during expansion. They *produce items*.

**Non-Production Orders**

1. Retainer Refresh: Goes to a Summoning Bell and refreshes retainer inventories. Useful in case you've moved stuff around, otherwise Lisbeth might have outdated inventory data.
2. Desynthesize: Desynthesizes an inventory item until you run out of them.
3. Spiritbind: Spiritbinds sets of gear by crafting fodder and rotating equipment.
4. Convert Materia: Converts an inventory item into materia until you run out of them.

These orders are not considered to produce anything, and so will never be created as suborders. You can consider them more like "tasks". Of them, only Spiritbind is used in order expanion, to fulfill its requirements with suborders when needed.

# Masterpiece Orders
Craft and gather masterpiece orders are special and executed differently. For craft masterpiece orders, Lisbeth expands them and creates collectable craft suborders to fulfill them. It calculates how many scripts these item turn ins give you, and uses that to calculate amounts. If you have the collectable items on inventory, they are "consumed" during expansion like anything else.

Gather masterpiece orders work differently. During expansion, Lisbeth only cares to know if your character is capable of collectable gathering any of the turn in items that fulfill the order. If so, it leaves the decision of which items to gather for later. During execution of orders, it will consider ALL unspoiled collectable gather turn ins and schedule them out. It will continue to gather these nodes until it has enough turn-ins to fulfill the masterpiece order. In other words, craft masterpiece suborders are calculated during expansion, while gather masterpiece suborders are dynamically generated during execution instead.

Both masterpiece orders are executed in batches, forming a mini-cycle of **batch gather/craft turn ins** => **turn them in** => **purchase stuff with scripts**. This is so the script cap is not a limitation, and allows you to make orders that go beyond the cap. The batch size is calculated during execution, based on the **expected** reward for turn ins. It calculates how many turn ins it can give before reaching the cap, and sets that as the batch size (with a maximum of 20 turn ins per batch, for inventory space purposes). So if an item gives 200 scripts, and the cap is 2000, the batch size will be 2000/200 = 10. However if the item gives 300 scripts, then it's 2000/300 = 6 since it's rounded down to avoid going above cap. The only time when it will go above cap is first if the expected script reward is different from the real one (maybe collectability is different than expected, or the item is starred), or secondly if there are no more incomplete *script consumers* (exchange orders that purchase stuff with scripts), or you asked for exactly 2000 scripts on your primary order, so it has to turn in a half-wasted item at the end to reach that amount exactly.

# Unspoiled Scheduling & GP Management
Timed node orders take priority over any other kind of order, and will **interrupt** other orders when a node spawns. This allows Lisbeth to do other things in between nodes and not be idle. As an example, it could be crafting something, and when the desired unspoiled node spawns it would stop crafting and go get the node, then returns to craft some more (or maybe do some other kind of order).

Lisbeth decides which unspoiled nodes to use (and which time slots) during execution, depending on several circumstances. The primary circumstance is GP management. When scheduling unspoiled nodes, it creates a sort of "timeline" of your GP using the regeneration per minute. It then assigns a gathering macro to each unspoiled order based on conditionals, and looks at the macro's **cost**. If the order is set for collectable gathering, the macro's cost is **mandatory**, which means it will skip a node spawn if it can't get enough GP by the time the node despawns because it would be a waste of travel time. If the order is not collectable, the macro's cost is optional. This means even if it doesn't have enough GP to use the macro, it'll go to the node anyways and gather without skill usage.

During GP scheduling, Lisbeth gives priority to orders as follows:

1. Unspoiled collectable orders.
2. Ephemeral orders.
3. Unspoiled ForceHQ orders.
4. Unspoiled orders.
5. Normal node collectable orders.
6. Normal node orders.

This means it "inserts" the orders into the schedule based on this priority, and when doing so the GP timeline changes based on the order's macro cost. If the schedule doesn't allow inserting a collectable order at the next node time, it skips it. But since collectables are given priority, this would only happen when the nodes spawn too close to one another, and neither cordials nor GP regen is enough to get enough GP for the second node's macro cost.

Normal node orders are lowest in GP priority, which means when there's unspoiled orders in the mix, normal node orders can only use skills when the schedule lets them. It's like the schedule telling them "Hey, you can use this much GP right now, but not more or I won't have enough for the next unspoiled node as planned". If that amount of GP is enough for skills, they'll get used. If not, they won't get used. So it might happen that you see your character with full GP gathering normal nodes, and wonder why its not using skills; and the reason would be that there's an unspoiled node coming up soon and Lisbeth is saving GP for that node. This GP management calculations take into consideration cordial gains and cooldowns.

## Gathering Macros

Lisbeth V4 changed gather macro usage a bit compared to previous versions. This time around, a macro is always used on unspiled nodes. If it's collectable, it's cost is mandatory; otherwise it's optional (but given GP priority as described above). Macros act as overrides similar to craft ones, where the macro whose conditions are met is chosen, and if several of them are available then the one with the highest GP requirement is used (since we assume the higher the macro's cost, the more yield/collectability it gives, the better). Lisbeth ships with default gathering macros for each circumstance (collectable, ephemeral, HQ, yield, etc), but you can make your own if you know what you're doing or wish to experiment.


# Craft Macros
Here you'll find a collection of Lisbeth macros that several community members have created. Although Lisbeth has a general crafting AI integrated, these macros might prove useful for specific circumstances.

Macros are a text file with a list of skills and conditionals that Lisbeth executes, and they can be used for both crafting and collectable gathering rotations.

Macros can be used in two ways: 

1. You can assign them directly to an order through that order's individual settings. This forces Lisbeth to use that macro when performing that order, regardless of priority or usage conditionals.
2. You can use them as **overrides**. When Lisbeth crafts something or gathers collectables, it will first check every enabled macro that you have to see if any of their usage conditionals are valid for the craft that is about to happen. If it finds one, it'll use that instead of the default crafting AI. In other words, it *overrides* the default behavior when the conditionals in the macro are met. This happens with primary and secondary orders (suborders). Only enabled macros are used in this manner.

## Macro Syntax

Macros are text files that reside in the **/Rebornbuddy/Settings/Lisbeth/Macros** folder. The files can be named anything you want, as long as their name is unique within the folder. Below you can see an example of a macro:

```
Name: Level 70 3Star 35 Durability - Specialist
Type: Craft
Enabled: False
Conditions: (MaxDurability == 35 && MaxProgress == 2365 && Craftsmanship >= 1500 && Control >= 1350 && Cp >= 524 && RecipeLevel == 350 && Specialized == True)

S1:
- Initial Preparations
- Comfort Zone
- Inner Quiet
- Specialty Reflect
- Steady Hand II
- Manipulation II
- Piece by Piece
- Piece by Piece
- Prudent Touch
- Prudent Touch
- Observe
- Focused Synthesis
- Comfort Zone
- Ingenuity
- Steady Hand II
- Innovation
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Steady Hand II
- Prudent Touch
- Ingenuity II
- Innovation

S2: (IsExcellent == true)
- Byregot's Blessing
- Observe
- Focused Synthesis

S3:
- Great Strides
- Byregot's Blessing
- Observe
- Focused Synthesis
```

Here is what the macro's properties do:

* **Name:** This is the name that will show on Lisbeth's UI. Try keep it the same as the file name for consistency; although this is not a requirement. 
* **Type:** Tells Lisbeth if the macro is for crafting or gathering.
* **Enabled:** This enables the macro as an override. You don't need to enable the macro to use it directly on an order's individual settings.
* **Conditions:** These conditions are checked when searching for a macro to use it as an override. Conditions can only have logical AND (&&) statements and must be inside parentheses. 

 **NOTE** Specialty skills should not be typed with the ":" symbol. Just remove that symbol from their name.

## Labels

As you can see from the example above, you can write multiple **segments** with skills, and give each one a label (i.e. S1, S2). Right beside the label, on the same line, you can add a conditional that uses the same syntax as the macro's general conditional property:

```
S2: (IsExcellent == true)
```
Lisbeth will execute the macro one **label** after the other, as in S1 => S2 => S3. On each label, it'll choose to use the first segment whose conditional is met. It only uses one segment per label (or none if no segments for that label had their conditional met). This means the relationship between segments with the same label is a logical OR.

*For S2, try use this segment first. If the condition is met, execute the skills and proceed to label S3. If the condition is not met, then try this other segment instead.*

## Jumps

Instead of using conditionals on the labels, you can also do jumps at the end of a label. For example:

```
S1:
- Initial Preparations
- Comfort Zone
- Inner Quiet
- Specialty Reflect
- Steady Hand II
- Manipulation II
- Piece by Piece
- Piece by Piece
- Prudent Touch
- Prudent Touch
- Observe
- Focused Synthesis
- Comfort Zone
- Ingenuity
- Steady Hand II
- Innovation
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Steady Hand II
- Prudent Touch
- Ingenuity II
- Innovation
=> S2 (IsExcellent == true)
=> S3

S2:
- Byregot's Blessing
- Observe
- Focused Synthesis

S3:
- Great Strides
- Byregot's Blessing
- Observe
- Focused Synthesis
```

## Jumps

You can also put conditionals in a line to pick the skill for that step. For example:

```
S1:
- Muscle Memory
- Comfort Zone
- Inner Quiet
- Steady Hand II
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Manipulation II
- Steady Hand II
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Prudent Touch
- Careful Synthesis II
- Steady Hand
- Great Strides
- (IsGoodOrExcellent == true) Tricks of the Trade | Ingenuity II
- Byregot's Blessing
- Careful Synthesis III
- Careful Synthesis III
- (Cp >= 7) Careful Synthesis III | Careful Synthesis II
- (Cp >= 7) Careful Synthesis III | Careful Synthesis II
```

**NOTE:** Whatever skill the macro selects on each step must be castable at that moment or Lisbeth will stop. In other words, Lisbeth assumes that all the skills the macro gives it as it executes are skills you want to cast, and if one of them doesn't get cast it considers it an error.

## Crafting Variables

These are all the variables you can use inside crafting macro conditionals. If there's a variable that's missing and you think it's useful let me know and I can add it.

```
int Recipe 
int RecipeLevel 
int RecipeDisplayLevel 
int RealLevelDifference 
int DisplayedLevelDifference 

int PlayerLevel 
int PlayerBaseLevel 
bool Specialized 
int Craftsmanship 
int Control 

int Cp 
int MaxCp 
int Progress 
int MaxProgress 
int Quality 
int MaxQuality 
int Durability 
int MaxDurability

int Step 
int HqPercent 

bool IsGood 
bool IsExcellent 
bool IsPoor 
bool IsNormal 
bool IsGoodOrExcellent

bool CanUseReuse
bool IsSuborder

int SteadyHandAura
int SteadyHand2Aura
int InnerQuietAura
int ComfortZoneAura
int MakersMarkAura
int WasteNotAura
int WasteNot2Aura
int GreatStridesAura
int InnovationAura
int ManipulationAura
int Manipulation2Aura
int IngenuityAura
int Ingenuity2Aura
int InitialPreparationsAura

int RemainingCp (cp + CZ stacks * 8)
```

## Gathering Macros

Gathering macros are used for collectable rotations. They have the same syntax as crafting macros, the only difference is the variables and skills you can use. Here's an example of one:

```
Name: Yellow Scripts 600GP
Type: Gather
Enabled: True
Conditions: (MaxGp >= 600)

S1:
- Discerning Eye
- Impulsive Appraisal II
- (DiscerningEyeAura > 0) Single Mind | Discerning Eye
- Impulsive Appraisal II
- (DiscerningEyeAura > 0) Single Mind | Discerning Eye
- Methodical Appraisal
- (Rarity < 470) Methodical Appraisal
```
## Gathering Variables

These are all the variables you can use inside gathering macro conditionals. If there's a variable that's missing and you think it's useful let me know and I can add it.

```
int PlayerLevel
int Gathering
int Perception
int MaxGp
int Gp
int Item

int SwingsRemaining
int MaxSwings
int Rarity
int Wear

int DiscerningEyeAura
int SingleMindAura
```