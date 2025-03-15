# Occurify.Astro

Astronomical instants and periods for Occurify: Track sun states, perform calculations, and manage events.

## 📖 Table of Contents  
- [Overview](#Overview)
- [Installation](#installation)
- [Usage](#usage)
- [Coordinates](#coordinates)
- [Potential Use Cases](#potential-use-cases)
    - [Get Sun Phase With Offset](#get-sun-phase-with-offset)
    - [Create Periods](#create-periods)
    - [Period Calculations](#period-calculations)
- [License](#license)

## Overview

- Uses the *SunCalcNet* library to enable functionality that:
- Supports location (coordinate) based instants and periods (e.g. dawn, daytime, etc).
- Supports multiple solar phases (sunrise, sunset, end of sunrise, start of sunset, (nautical) dawn, (nautical) dusk, (end of) night, (end of) golden hour, solar noon and nadir).

For details on the Occurify ecosystem including examples for this library, please check the following documentation: [Occurify main README](https://github.com/Occurify/Occurify).

## Installation

Occurify.Astro is distributed as a [NuGet package](https://www.nuget.org/packages/Occurify.Astro), you can install it from the official NuGet Gallery. Please use the following command to install it using the NuGet Package Manager Console window.
```
PM> Install-Package Occurify.Astro
```

## Usage

Rather than working with concrete instants and periods in time, Occurify allows for conceptual representation of time using instant and period timelines.

For example, rather than listing all sunrises of a year to work with, you can define the concept of "all sunrises", apply transformations or filters, and extract the relevant periods as needed.

The following example demonstrates how to define a timeline, `sunrises` that represents all sunrises:
```cs
ITimeline sunrises = AstroInstants.SunPhases(SunPhases.Sunrise);
```

We can use this timeline to determine the previous or next sunrise:

```cs
Console.WriteLine($"The previous sunrise was at {sunrises.GetPreviousUtcInstant(DateTime.UtcNow)}");
Console.WriteLine($"The next sunrise will be at {sunrises.GetNextUtcInstant(DateTime.UtcNow)}");
```

Or use it for more complicated logic:

```cs
Console.WriteLine(
    $"The past 100 days contained :{sunrises.EnumerateRange(DateTime.UtcNow - TimeSpan.FromDays(100), DateTime.UtcNow).Count()} sunrises.");
Console.WriteLine(
    $"The year 2050 will contain :{sunrises.EnumerateRange(new DateTime(2050, 1, 1).AsUtcInstant(), new DateTime(2051, 1, 1).AsUtcInstant()).Count()} sunrises.");
```

The timeline **only resolves the necessary instants when enumerated**, ensuring efficiency.

This approach allows developers to focus on what they need—such as "sunrises"—without manually managing time calculations. As a result, coding becomes more intuitive and use case-driven.

## Coordinates

All methods in `Occurify.Astro` have a signature with and without `Coordinates` object. If no `Coordinates` object is provided, the method will use `Coordinates.Local` by default. Note that this static property needs to be set before use:

```cs
Coordinates.Local = new Coordinates(48.8584, 2.2945);
```

## Potential Use Cases

This section presents various use cases that demonstrate Occurify’s capabilities and provide a clearer understanding of its functionality.

>**Note: Instead of using `var`, variable types are explicitly defined in the examples for improved clarity.**

### Get Sun Phase With Offset

```cs
ITimeline hourAfterSunrises = AstroInstants.SunPhases(SunPhases.Sunrise) + TimeSpan.FromHours(1);
DateTime? nextOccurrance = hourAfterSunrises.GetNextUtcInstant(DateTime.UtcNow);
```

#### Using ReactiveX for Scheduling

Now we can use `ToObservable` from `Occurify.Reactive` to integrate with `ReactiveX` for event-driven scheduling:
```cs
hourAfterSunrises.ToObservable(scheduler).Subscribe(dateTime =>
{
    Console.WriteLine($"It's {dateTime} and the sun is already up for 1 hour!");
});
```

### Create Periods

Occurify allows us to combine two timelines into a period timeline:

```cs
ITimeline sunrises = AstroInstants.SunPhases(SunPhases.Sunrise);
ITimeline sunsets = AstroInstants.SunPhases(SunPhases.Sunset);
IPeriodTimeline dayTimes = sunrises.To(sunsets);
```

We can sample today:

```cs
Period daytimeToday = sunrises.EnumeratePeriod(TimeZonePeriods.Today()).First(); // Note that this could return no periods depending on your location. For example on the Arctic.
Console.WriteLine($"Daytime today is {daytimeToday.Duration} long.");
```

But are free to sample other periods:

```cs
TimeSpan? duration = dayTimes.EnumeratePeriod(TimeZonePeriods.Year(2050)).TotalDuration();
Console.WriteLine($"In 2050, the sun is up for a total of {duration}.");
```

### Period Calculations

The following example illustrates how we can calculate the number of days in the current year (using `Occurify.TimeZones`) in the Arctic region that do not experience either a sunset or a sunrise:

```cs
Coordinates arcticCoordinates = new Coordinates(80.45302, 54.77918, Height: 37);
ITimeline sunsetsAndRises = AstroInstants.SunPhases(arcticCoordinates, SunPhases.Sunrise | SunPhases.Sunset);
IPeriodTimeline daysOfCurrentYear = TimeZonePeriods.Days().Within(TimeZonePeriods.CurrentYear());
IPeriodTimeline daysWithoutSunsetsOrRises = daysOfCurrentYear - daysOfCurrentYear.Containing(sunsetsAndRises);

Console.WriteLine($"This year on the arctic the sun doesn't rise or set on {daysWithoutSunsetsOrRises.Count()} days.");
```

## License

Copyright © 2025 Jasper Lammers. Occurify.Astro is licensed under [The MIT License (MIT)](https://github.com/Occurify/Occurify.Astro?tab=MIT-1-ov-file).