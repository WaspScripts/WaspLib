# WaspLib

WaspLib is a library for writing Old School RuneScape scripts with Simba 2.0.
Simba runs Lape scripts (Lape is Pascal-like). WaspLib lives at
`Includes/WaspLib` inside a Simba install.

Run every command below from the Simba directory, the one that contains the
Simba executable. The examples use `./Simba`; on Windows use `Simba.exe`.

## Compile and run

```
./Simba --compile path/to/script.simba
./Simba --target=<handle> --run path/to/script.simba
```

- `--compile` only compiles. On failure it prints one line with the error,
  line, column and file, e.g.
  `Unknown declaration "Foo" at line 2, column 11 in file "..."`.
- `--run` runs headless, with output on stdout. Without `--target` the target is
  the whole desktop.
- Headless scripts have `SIMBAHEADLESS` defined. `GetSimbaTargetWindow()`
  needs the IDE, so it doesn't work there.

## Finding the game client

```
./Simba --run Includes/WaspLib/tools/agents/find_client.simba
```

This prints one tab-separated line per open client:
`<handle>  <client>  <pid>  <title>`. A RuneLite title includes the
character name, which tells clients apart. The handle points at the game canvas,
the window a script should target.

## Working with the client

There are two preludes in `tools/agents/`. Pick one, write a short snippet that
includes it, and run the snippet against the client handle.

| Prelude | Snippets go in | Input to the client |
|---|---|---|
| `probe.simba` | `Agents/probes/` | None. Read-only. |
| `act.simba` | `Agents/actions/` | Yes, through fake input. |

Both give you `TAgent.Save(img, name)`, which writes `Agents/<name>.png` and
prints the path. Open the image to look at it. Draw on the image first
(`DrawBox`, `DrawTPA`, `DrawText`, ...) to check what a finder matched.
`Agents/` holds scratch files only, so overwrite files there freely.

A run takes about 3 seconds. Interface calls made while logged out may wait on
interfaces that aren't there, so check `Client.IsLoggedIn()` first.

### Probing

Use a probe to see or measure something. Probes include WaspLib with
`WL_READ_ONLY`, so they never send input to the client and are safe while a bot
runs on it.

```pascal
{$I WaspLib/tools/agents/probe.simba}
begin
  WriteLn Inventory.Slots.Count();
  TAgent.Save(Target.GetImage(), 'client');
end.
```

```
./Simba --target=<handle> --run Agents/probes/<name>.simba
```

A probe snippet must include `probe.simba` and nothing that sends input.
Nothing enforces this: the file's contents decide what it does, not its folder.
Users may let probes run without asking them, so keep to the rule.

### Acting

Use an action only when the user asks you to do something in the game.
`act.simba` sets WaspLib up like a normal script:

- Fake input goes into the client without moving the user's real mouse.
- It can't run on a client that a script is already botting on.
- When logged in, WaspLib's setup may adjust the camera, brightness, zoom and
  roofs first.

```pascal
{$I WaspLib/tools/agents/act.simba}
begin
  if not Client.IsLoggedIn() then
    Login.DoLogin();
  Inventory.Open();
  TAgent.Save(Target.GetImage(), 'client');
end.
```

```
./Simba --target=<handle> --run Agents/actions/<name>.simba
```

- `Login.DoLogin()` uses the user's saved WaspLib profile. Never ask for,
  write or print account credentials.
- Keep each action small, then check the result with a probe or a saved image
  before the next one.

## Looking up APIs

- Simba built-ins: `Data/Dumps/` holds Simba's declaration dump (plain Pascal
  declarations, so grep it). Simba's source is at
  https://github.com/Villavu/Simba; the script imports are
  `Source/script/imports/simba.import_*.pas`.
- WaspLib: each method has a doc comment with an example just above it.
  `main.simba` and `utils.simba` list every file in include order.
- Before using a WaspLib method, grep for it. Don't guess names: for example,
  there is no `Inventory.Count()`. It's `Inventory.Slots.Count()` or
  `Inventory.Items.Count(item)`.

## Code style

- Match the surrounding code: 2-space indent, `Self.` for fields, records
  with methods instead of classes.
- Constants belonging to a record go on the record
  (`const NAME: Type = value;`). A constant used by one function goes in that
  function's `const` section. A literal used once or twice stays inline.
- Keep code lean. Don't guard against states that can't happen, and don't
  re-initialise values Simba already defaults (a new `TImage` is already
  black).
