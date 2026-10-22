```mermaid
classDiagram
    Describable <|--Entity
    Flammable <|--Goblin
    Effect <|-- Poison
    Effect <|-- Regen
    Item <|-- Key
    Item <|-- Weapon
    Item <|-- Consumable
    Weapon <|--Sword
    Weapon <|--Bow
    Consumable <|-- Potion
    Entity  <|-- Creature
    Creature <|-- Player
    Creature <|-- Monster
    Monster <|-- Goblin
    Monster <|-- Skeleton

```