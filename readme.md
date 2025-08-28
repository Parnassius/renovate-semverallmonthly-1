# 37753

Reproduction for https://github.com/renovatebot/renovate/discussions/37753

## Current behavior

The [`config:semverAllMonthly`](https://docs.renovatebot.com/presets-config/#configsemverallmonthly) preset only updates direct dependencies, but doesn't regenerate lockfiles.

## Expected behavior

Lockfiles (for example `uv.lock`) should be regenerated and indirect dependencies should get updated.

## Root cause

The `config:semverAllMonthly` preset expands to this:
```json
{
  "extends": [
    ":preserveSemverRanges",
    "group:all",
    "schedule:monthly",
    ":maintainLockFilesMonthly"
  ],
  "lockFileMaintenance": {
    "commitMessageAction": "Update",
    "extends": [
      "group:all"
    ]
  },
  "separateMajorMinor": false
}
```

The issue seems to stem from the `group:all` preset inside the `lockFileMaintenance` section. Expanding that leads to the following configuration:
```json
{
  "extends": [
    ":preserveSemverRanges",
    "group:all",
    "schedule:monthly",
    ":maintainLockFilesMonthly"
  ],
  "lockFileMaintenance": {
    "commitMessageAction": "Update",
    "groupName": "all dependencies",
    "groupSlug": "all",
    "lockFileMaintenance": {
      "enabled": false
    },
    "packageRules": [
      {
        "groupName": "all dependencies",
        "groupSlug": "all",
        "matchPackageNames": [
          "*"
        ]
      }
    ],
    "separateMajorMinor": false
  },
  "separateMajorMinor": false
}
```

If I remove the nested `lockFileMaintenance` the bot starts creating pull requests which include lock file maintenance, see this second example repository: https://github.com/Parnassius/renovate-semverallmonthly-2
