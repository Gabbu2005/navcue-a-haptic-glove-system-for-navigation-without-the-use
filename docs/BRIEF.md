# The brief

Describe the board you want: what it does, the parts it must use, the
interfaces it exposes, and the constraints that bound the layout.

A hosted `create` run overwrites this file with the brief you typed in the
console, then works from it.

## What makes a brief usable

- **Parts** by designator where you care, and by function where you do not.
- **Power**: what comes in, what rails come out, and how much current.
- **Interfaces**: every connector and bus, and what is on it.
- **Constraints**: layer count, board outline, assembly method, and anything
  the board must not do.
