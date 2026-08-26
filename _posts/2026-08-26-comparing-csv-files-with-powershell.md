---
layout: post
title:  "An Interesting Use of PowerShell: Comparing CSV and Delimited Files"
author: "Sagiv Barhoom"
date:   2026-08-26
categories: PowerShell
background: '/img/posts/bg-shells.jpg'
---

# An Interesting Use of PowerShell: Comparing CSV and Delimited Files

Not long ago, a developer working on a migration from SQR to PL/SQL asked me for help.

One of the challenges was verifying that the new code produced the same data as the original SQR program. Both versions generated directories containing CSV-like text files, with fields separated by `|`.

This turned into a useful PowerShell exercise: compare two sets of delimited files and produce a focused report containing only the differences.

## What We Needed to Compare

We needed to verify two things:

1. Every file in the old output also existed in the new output, and vice versa.
2. For files that existed on both sides, identify only the records that appeared on one side but not the other.

The goal was not to dump both outputs into a large report.

We wanted the tester to see exactly where the differences were instead of manually reviewing dozens of files and thousands of records.

## Why PowerShell?

The developer mainly works with Oracle, APEX, and SQL Server, and less with languages such as Python, Perl, or Bash.

The target environment was Windows 11, so PowerShell was a natural choice. It was already available on the target machine, required no additional runtime for this workflow, and the script could be executed directly in the same environment where the migration was being tested.

Because text encoding behavior differs between Windows PowerShell and newer cross-platform PowerShell versions, the script was validated using the local PowerShell environment on the target Windows 11 machine rather than assuming that behavior observed elsewhere would be identical.

The interesting part was how little code was required to perform a useful comparison.

## Comparing Complete Rows

For this specific reconciliation, we did not need to understand the meaning of every column.

A record was considered equal when the complete output line was equal.

For example, consider these two files.

`before.txt`:

```text
StudentId|StudentName|Email
1001|Alice Brown|alice@example.com
1002|David Green|david@example.com
1003|Maria Stone|maria@example.com
```

`after.txt`:

```text
StudentId|StudentName|Email
1001|Alice Brown|alice@example.com
1002|David Green|david@example.com
1004|John White|john@example.com
```

The relevant differences are:

```text
Only in before:
1003|Maria Stone|maria@example.com

Only in after:
1004|John White|john@example.com
```

The script reads each line and stores the number of times it appears. The `Encoding` parameter is deliberately explicit because legacy migration outputs may depend on the Windows code page; the correct value should match the encoding of the files being reconciled:

```powershell
function Get-LineMultiset {
    param(
        [string]$Path,
        [string]$Encoding = "Default"
    )

    $counts =
        [System.Collections.Generic.Dictionary[string,int]]::new(
            [System.StringComparer]::Ordinal
        )

    foreach ($line in (Get-Content -LiteralPath $Path -Encoding $Encoding)) {
        if ($counts.ContainsKey($line)) {
            $counts[$line]++
        }
        else {
            $counts[$line] = 1
        }
    }

    return $counts
}
```

The dictionary uses an ordinal, case-sensitive comparison. If the case of a value changes, the script treats it as a real difference.

It also keeps a count for each line instead of storing only a unique set of rows. This is important when duplicate records exist.

If the same row appears three times in one file and twice in the other, the difference is one row.

The comparison itself is straightforward:

```powershell
function Get-MultisetDifference {
    param(
        [System.Collections.Generic.Dictionary[string,int]]$Left,
        [System.Collections.Generic.Dictionary[string,int]]$Right
    )

    foreach ($line in $Left.Keys) {
        $leftCount = $Left[$line]
        $rightCount = 0

        if ($Right.ContainsKey($line)) {
            $rightCount = $Right[$line]
        }

        $difference = $leftCount - $rightCount

        if ($difference -gt 0) {
            [PSCustomObject]@{
                Line  = $line
                Count = $difference
            }
        }
    }
}
```

And then:

```powershell
$before = Get-LineMultiset -Path ".\before.txt"
$after  = Get-LineMultiset -Path ".\after.txt"

$onlyBefore = @(
    Get-MultisetDifference -Left $before -Right $after
)

$onlyAfter = @(
    Get-MultisetDifference -Left $after -Right $before
)
```

This has two useful properties for this type of reconciliation:

- Row order does not matter.
- Duplicate row counts do matter.

## First Check: Are the Files Already Identical?

Before comparing individual rows, the script calculates a SHA256 hash for each file:

```powershell
$beforeHash = (Get-FileHash -LiteralPath $beforeFile -Algorithm SHA256).Hash
$afterHash  = (Get-FileHash -LiteralPath $afterFile -Algorithm SHA256).Hash

if ($beforeHash -eq $afterHash) {
    "IDENTICAL"
}
```

If the SHA256 hashes match, the script treats the files as byte-for-byte identical and there is no reason to perform a row comparison.

If the hashes differ, the script continues with the multiset comparison.

This also lets the script distinguish between two cases:

- `IDENTICAL`: the files are byte-for-byte identical.
- `LOGICALLY_IDENTICAL`: the file hashes differ, but both files contain the same rows with the same number of occurrences.

The second case can happen, for example, when the same rows appear in a different order.

## Why We Did Not Use `Import-Csv`

PowerShell has a built-in command for reading CSV and other delimited files:

```powershell
$data = Import-Csv -Path $file -Delimiter '|'
```

So why did the comparison script use `Get-Content` instead?

Because the immediate requirement was to compare complete output records, not to interpret each file as a table.

For the reconciliation, this row:

```text
1002|David Green|david@example.com
```

is simply one value that either exists in both outputs or does not.

The comparison does not need to know that `1002` is a student ID or that the third value is an email address.

The same idea applies when doing a quick manual spot check. If I want to locate a specific raw record in a file, `Select-String` is enough:

```powershell
Select-String -LiteralPath $file -Pattern "1002" -SimpleMatch |
    Select-Object -ExpandProperty Line
```

This behaves much like a simple `grep` search: `-SimpleMatch` treats the pattern as literal text rather than a regular expression, and expanding the `Line` property returns the matching line itself instead of the surrounding `MatchInfo` object.

For this task, treating the record as a raw line also made the comparison independent of column names.

## When `Import-Csv` Is the Better Choice

`Import-Csv` becomes more useful when the fields themselves matter.

Suppose the file looks like this:

```text
StudentId|StudentName|Email|Mobile
123456789|Alice Brown|alice@example.com|555-0101
987654321|David Green|david@example.com|555-0102
```

PowerShell can parse it directly. This example assumes that the first line of the file contains column headers:

```powershell
$data = Import-Csv -LiteralPath ".\students.txt" -Delimiter '|'

$row = $data | Where-Object {
    $_.StudentId -eq '123456789'
}

$row
```

If the file does not contain a header row, `Import-Csv` can still be used by supplying column names explicitly with the `-Header` parameter.

The result is an object whose properties come from the header:

```text
StudentId   : 123456789
StudentName : Alice Brown
Email       : alice@example.com
Mobile      : 555-0101
```

Individual values can then be accessed directly:

```powershell
$row.StudentName
$row.Email
$row.Mobile
```

This is much better if the requirement is to find a record by a key such as `StudentId` and compare specific fields between two versions.

For example, assuming `StudentId` uniquely identifies a single record:

```powershell
$beforeData = Import-Csv -LiteralPath ".\before.txt" -Delimiter '|'
$afterData  = Import-Csv -LiteralPath ".\after.txt"  -Delimiter '|'

$beforeRow = $beforeData | Where-Object {
    $_.StudentId -eq '123456789'
}

$afterRow = $afterData | Where-Object {
    $_.StudentId -eq '123456789'
}
```

Now we can compare the fields that matter:

```powershell
$fields = 'StudentName', 'Email', 'Mobile'

foreach ($field in $fields) {
    if ($beforeRow.$field -ne $afterRow.$field) {
        [PSCustomObject]@{
            Field  = $field
            Before = $beforeRow.$field
            After  = $afterRow.$field
        }
    }
}
```

For example, if only the email address changed, the output could be:

```text
Field  Before                After
-----  ------                -----
Email  alice@example.com     alice.new@example.com
```

So the choice depends on what is being compared:

- `Get-Content` is useful when the complete row is the unit of comparison.
- `Select-String` is useful for locating raw text in a file.
- `Import-Csv` is useful when the file structure and individual columns matter.

## What the Final Script Produces

The final script compares two output directories and generates HTML reports.

It identifies:

- Files that exist only on one side.
- Files that are byte-for-byte identical.
- Files that contain the same rows but in a different order.
- Files with actual row differences.
- Rows that exist only in the old output.
- Rows that exist only in the new output.

For files with differences, the report contains only the rows that require attention.

This is much easier to review than comparing complete output files manually.

## Using AI to Build the Tool

We used ChatGPT to create the first version of the PowerShell script from the requirements and the file structure.

The first version was not perfect.

We ran it, found problems, corrected them, and repeated the process until the script became useful for the actual migration.

This is where I found the AI particularly useful: not as the comparison engine, but as a way to get from a concrete requirement to a working utility without first becoming a PowerShell expert.

## Why Not Let AI Compare the Files Directly?

Another option would have been to give both output directories to an AI model after every migration run and ask it to identify the differences.

For this type of process, I prefer the comparison itself to remain deterministic.

Migration testing is repetitive:

```text
change the PL/SQL code
generate new output
compare
fix
generate output again
compare again
```

For each run, I want the same inputs and the same comparison rules to produce the same result.

So the division of responsibility was simple:

**Generative AI helped build and modify the tool. Deterministic PowerShell code performs the repeated comparison.**

## Testing in the Target Environment

An important limitation appeared when we tried to let the AI execute the PowerShell script itself.

The AI attempted to run the script using `pwsh` inside a Linux-based container. The script, however, was intended for the Windows 11 environment where the migration testing was actually being performed.

The AI was not able to validate the script successfully in that Linux `pwsh` environment. Because of that, I stopped relying on the AI to perform the execution tests.

I continued using AI to help write, review, and correct the PowerShell code, but I ran and validated every version myself on the target Windows 11 machine.

This distinction became important during the work: the AI was useful for developing the utility, but the actual validation had to be performed in the same environment in which the script would be used.

## Reusing the Approach

The script was created for a specific SQR-to-PL/SQL migration, but the comparison approach is not tied to SQR or PL/SQL.

The same basic logic can be reused for:

- CSV files.
- Pipe-delimited files.
- Other text-based structured output.
- Migration reconciliation.
- Regression checks where row order should be ignored.
- Field-level comparisons by switching to `Import-Csv` when the data structure matters.

The useful part is choosing the right level of comparison: raw rows when the whole record matters, parsed objects when individual fields matter.
