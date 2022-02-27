# Window

A go implemetnation of a Window - a data set that collects events together.

Bring your own events - each one needs to have a unique ID and a (not necessarily unique) timestamp. 

You can add events to the window; you can remove them by ID. You can trim the
"back" of the window, throwing out all events older than a specified date. 

You can marshal the window to json, protobuf and avro; you can unmarshal from
those formats, too.

You can archive the window to CSV or parquet. 

You can iterate through the events in a window, optionally specifying
sub-windows.

# Getting Started

Create a new window:

```
w := NewWindow()
```

Define your events by implementing the Event interface:

```
type MyEvent struct {
  id uuid.UUID
  t time.Time
  v string
}

func (e *MyEvent) GetID() uuid.ID {
  return e.id
}

func (e *MyEvent) GetTimestamp() time.Time {
  return e.t
}
```

Add an event; errors if the ID exists with the same timestamp:

```
event := MyEvent{
  id: uuid.New()
  t: time.Now()
  v: "hi there!"
} 

err := w.Add(event)
```
Note that if you re-use the ID with different timestamps they're all going to
be stored. If you delete the event by ID, only the earliest event is going to
be deleted. Don't re-use your IDs!

Remove an event; errors if the ID's not found:

```
err := w.Del(id)
if err != nil {
  log.Fatal(err)
}
```
Note this is expensive; we have to go hunting for the event.

Trim a window:

```
// throw out events earlier than 24 hours ago
since := time.Now().Add(-time.Duration(24*time.Hour)) 
w.Trim(since)
```

Iterate over a window:

```
iterator := w.Iterator() // iterator starts at the earliest event
for iterator.Next() {
  e := iterator.event()
  // .. do stuff with e
}
err := iterator.Error()
if err != nil {
  log.Fatal(err)
}
```

Iterate over a lagged, subwindow:
```
// the window should be from 2 days ago to 1 day ago
since := time.Now().Add(-time.Duration(48*time.Hour)) 
delta := 24*time.Hour

opt := IteratorOps{
  start: since,
  width: delta,
}
iterator := w.Iterator(opt)
for iterator.Next() {
  e := iterator.event()
  // .. do stuff with e
}
err := iterator.Error()
if err != nil {
  log.Fatal(err)
}
```

Marshal to JSON:
```
jsonBytes, err := w.MarshalJSON()
if err != nil {
  log.Fatal(err)
}
```

Unmarshal JSON into a window:

```
jsonBytes, err := os.Readfile()
if err != nil {
  log.Fatal(err)
}
w := New(Window)
err := json.Unmarshal(jsonBytes, w)
if err != nil {
  log.Fatal(err)
}
```








