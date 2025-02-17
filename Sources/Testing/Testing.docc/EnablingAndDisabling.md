# Enabling and disabling tests

<!--
This source file is part of the Swift.org open source project

Copyright (c) 2023-2024 Apple Inc. and the Swift project authors
Licensed under Apache License v2.0 with Runtime Library Exception

See https://swift.org/LICENSE.txt for license information
See https://swift.org/CONTRIBUTORS.txt for Swift project authors
-->

Conditionally enable or disable individual tests before they run.

## Overview

Often, a test is only applicable in specific circumstances. For instance,
you might want to write a test that only runs on devices with particular
hardware capabilities, or performs locale-dependent operations. The testing
library allows you to add traits to your tests that cause runners to
automatically skip them if conditions like these are not met.

- Note: A condition may be evaluated multiple times during testing.

### Disable a test

If you need to disable a test unconditionally, use the
``Trait/disabled(_:fileID:filePath:line:column:)`` function. Given the following
test function:

```swift
@Test("Food truck sells burritos")
func sellsBurritos() async throws { ... }
```

Add the trait _after_ the test's display name:

```swift
@Test("Food truck sells burritos", .disabled())
func sellsBurritos() async throws { ... }
```

The test will now always be skipped.

It's also possible to add a comment to the trait to present in the output from
the runner when it skips the test:

```swift
@Test("Food truck sells burritos", .disabled("We only sell Thai cuisine"))
func sellsBurritos() async throws { ... }
```

### Enable or disable a test conditionally

Sometimes, it makes sense to enable a test only when a certain condition is met. Consider
the following test function:

```swift
@Test("Ice cream is cold")
func isCold() async throws { ... }
```

If it's currently winter, then presumably ice cream won't be available for
sale and this test will fail. It therefore makes sense to only enable it if it's currently summer. You can conditionally enable a test with
``Trait/enabled(if:_:fileID:filePath:line:column:)``:

```swift
@Test("Ice cream is cold", .enabled(if: Season.current == .summer))
func isCold() async throws { ... }
```

It's also possible to conditionally _disable_ a test and to combine multiple
conditions:

```swift
@Test(
  "Ice cream is cold",
  .enabled(if: Season.current == .summer),
  .disabled("We ran out of sprinkles")
)
func isCold() async throws { ... }
```

If a test is disabled because of a problem for which there is a corresponding
bug report, you can use the ``Trait/bug(_:_:)-2u8j9`` or
``Trait/bug(_:_:)-7mo2w`` function to show the relationship:

```swift
@Test(
  "Ice cream is cold",
  .enabled(if: Season.current == .summer),
  .disabled("We ran out of sprinkles"),
  .bug("12345")
)
func isCold() async throws { ... }
```

If a test has multiple conditions applied to it, they must _all_ pass for it to
run. Otherwise, the test notes the first condition to fail as the reason the test
is skipped.

### Handle complex conditions

If a condition is complex, consider factoring it out into a helper function to
improve readability:

```swift
func allIngredientsAvailable(for food: Food) -> Bool { ... }

@Test(
  "Can make sundaes",
  .enabled(if: Season.current == .summer),
  .enabled(if: allIngredientsAvailable(for: .sundae))
)
func makeSundae() async throws { ... }
```
