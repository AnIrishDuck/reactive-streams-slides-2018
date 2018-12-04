# Reactive Streams

---

## tl;dr

We can improve our code

By stealing ideas from functional programming.

---

### Objectifying Reality

Promises turn single-shot callbacks into first-class values:


    // from this:
    fs.readFile(path, (err, data) => console.log(data))

    // to this:
    fs.readFile(path)
        .then((data) => console.log(data))

---

### Objectifying Reality

Reactive streams convert streams of events into first-class values:


    // from this:
    el.onchange = (value) => { view.innerHTML = value }

    // to this:
    most.fromEvent(el, 'onchange')
        .observe((value) => { view.innerHTML = value })

---

### Object Permanence

- Futures are inherently "multi-cast"
- Functions that return futures are **composable**

---

## The Pyramid of Doom

    fetchUsers((err, ids) => {
        const id = findActiveUser(ids)
        postGrade(id, grade, () => {
            sendNotification(id, () => {
                screenReaderNotification()
            })
        })
    })

---

## Promises

    // time ->
    //
    // |--------------o
    //
    // |--------------x

---

## Chaining

    // time ->
    //
    // |-----o
    //       |-----o
    //             |-----o
    //                   |-----o

    fetchUsers
        .then((ids) => {
            const id = findActiveUser(ids)
            postGrade(id, grade)
        })
        .then((id) => sendNotification(id))
        .then(() => screenReaderNotification())

---

## Combining

    // time ->
    //
    // |---o
    // |-----o
    //       |--o

    Promise.all([fetchStudentIds(), fetchAssessmentId()])
        .then(([ids, assessmentId]) => fetchGrades(ids, assessmentId))
        .then((grades) => renderUI(grades))

---

## Leaky Abstractions
###### https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/

1. All abstractions are leaky.
2. Fixing the leaks get harder the higher we go.

---

## The Phantom Menace

    function loadData () {
        fetchData()
            .then((data) => doComplicatedTransform(data))
            .then((transformed) => renderResults(transformed))
    }

---

## Unmasked

    function loadData () {
        fetchData()
            .then((data) => doComplicatedTransform(data)) // boom!
            .then((transformed) => renderResults(transformed))
    }

---

## Banished

    function loadData () {
        // begone, phantom error!
        return fetchData()
            .then((data) => doComplicatedTransform(data))
            .then((transformed) => renderResults(transformed))
    }


---

## Reactive Streams

- Reactive streams can emit multiple values.
- "resolving" a stream is decoupled from "emitting" data values.

---

## Lifetimes of Reactive Streams

    // time ->
    //
    // |--a-b-c-d|
    //
    // |---a----------x
    //
    // |----a------------->

---

## Reactive Streams Subsume Promises

    // A promise that resolves is a stream that emits a single value and then
    // immediately resolves:
    // |------o
    // |------a|
    //
    // A promise and a stream can reject:
    // |-----x
    //
    // A promise and a stream can *never* resolve (beware of memory leaks!)
    // |------------------>

---

## Complex Stream Processing

Let's say you want to write some code that does live updates from a server:

- Wait for the user to input some data.
- Debounce the input.
- Make a request using that input as a data parameter.
- Display the relevant output.

---

## Simple?

    const doRequest = (query) => {
        request(query).then((result) => updateView(result))
    }

    input.onchange = _.debounce(doRequest)

---

## Too Soon!

    // input     |---a---b---------------->
    // request a     |-------------ra
    // request b         |---rb
    // view      |-----------rb----ra!---->

---

![bad things happened](//localhost:8001/narratives.png)

---

## Fixed

    let ix = 0
    const doRequest = (query) => {
        ix += 1
        const current = ix
        request(query)
            .then((result) => current === ix ? updateView(result) : null)
    }

    input.onchange = _.debounce(doRequest)

---

## Reactive

    most.fromEvent(input, 'onchange')
        .debounce()
        .flatMap((query) => most.fromPromise(request(query)))
        .switchLatest()
        .observe(updateView)

---

## Advantages

- Streams are first-class objects that can be returned.
- Streams can be shared (in most, via `.multicast()`)

---

## Disadvantages

- Error handling is where promises were five years ago.
- Streams sharing can do strange things (without `.multicast()`)
- "hot" versus "cold" observables.

---

## Stream Sharing

    const data = keyInputs.map((string) => parseInt(string) * 2)

    data.observe((x) => console.log(x))
    data.observe((x) => updateView(x))

---

## Sharing for Everyone

    const data = keyInputs.map((string) => parseInt(string) * 2)
                          .multicast()

    data.observe((x) => console.log(x))
    data.observe((x) => updateView(x))

---

## Missed Connections

    function gradeOnCurve () {
        const ee = new EventEmitter()
        const spy = sinon.spy((args) => ee.emit('call', args))
        const calls = most.fromEvent(ee, 'call')

        simulateThreeLoadCalls(spy)

        const firstArgs = calls
            .map((args) => args[0])
            .take(3)
            .reduce((acc, cur) => [...acc, cur])

        expect(firstArgs).toEqual(['call 1', 'call 2', 'call 3'])
    }

---

## Dude, Where's My Call

    describe('it calls the provided callback three times', () => {
        const ee = new EventEmitter()
        const spy = sinon.spy((args) => ee.emit('call', args))
        const calls = most.fromEvent(ee, 'call').take(3)
            .take(3)
            .reduce((acc, cur) => [...acc, cur])

        performThreeLoads(spy)

        const firstArgs = calls.map((args) => args[0])
        expect(firstArgs.reduce((acc, cur) => [...acc, cur]))
            .toEqual(['call 1', 'call 2', 'call 3'])
    })

---

## Plugging the Leaks

Reactive streams are a great way to wrap / build event emitters.

---

## Final Thoughts

- The Elephant in the Room
- The npm Way
- When in Rome

---

## The Elephant in the Room

---

## The Elephant in the Room

`event-stream`

---

## The Elephant in the Room

![the npm event-stream library has over one million weekly downloads](//localhost:8001/event-stream.png)

---

## The Elephant Sneaks Into the Room

`not@black.hat`


###### https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident

---

## The Elephant is Now Rampaging Around the Room

![bad things happened](//localhost:8001/fail.jpg)


---

## The npm Way

- most (going 2.0, hopefully soon)
- RxJS
- bacon

---

## When in Rome

- Reactive streams are pretty cool.

---

## When in Rome

- Reactive streams are pretty cool.
- Get everybody on board **before** using them.

---

## Any Questions?

![questions?](//localhost:8001/david-pumpkins.jpg)
