# Occurify.Astro

Astronomical instants and periods for Occurify: Track sun states, perform calculations, and manage events.

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

Rather than working with concrete instants and periods in time, Occurify allows for conceptual representation of time using intstant and period timelines.

For example, rather than listing all sunrises of a year to work with, you can define the concept of "all sunrises", apply transformations or filters, and extract the relevant periods as needed.

The following example illustrates how we can calculate the number of days in the current year in the Arctic region that do not experience either a sunset or a sunrise:

```cs
Coordinates arcticCoordinates = new Coordinates(80.45302, 54.77918, Height: 37);
ITimeline sunsetsAndRises = AstroInstants.SunPhases(arcticCoordinates, SunPhases.Sunrise | SunPhases.Sunset);
IPeriodTimeline daysOfCurrentYear = TimeZonePeriods.Days().Within(TimeZonePeriods.CurrentYear());
IPeriodTimeline daysWithoutSunsetsOrRises = daysOfCurrentYear - daysOfCurrentYear.Containing(sunsetsAndRises);

Console.WriteLine($"This year on the arctic the sun doesn't rise or set on {daysWithoutSunsetsOrRises.Count()} days.");
```

The period timeline **only resolves the necessary periods when enumerated**, ensuring efficiency.

This approach allows developers to focus on what they need—such as "sunrises"—without manually managing time calculations. As a result, coding becomes more intuitive and use case-driven.

## Coordinates

All methods in `Occurify.Astro` have a signature with and without `Coordinates` object. If no `Coordinates` object is provided, the method will use `Coordinates.Local` by default. Note that this static property needs to be set before use:

```cs
Coordinates.Local = new Coordinates(48.8584, 2.2945);
```

## License

Copyright © 2025 Jasper Lammers. Occurify.Astro is licensed under The MIT License (MIT).