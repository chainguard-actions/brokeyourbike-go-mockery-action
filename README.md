# go-mockery-action

[![Latest Stable Version](https://img.shields.io/github/v/release/brokeyourbike/go-mockery-action)](https://github.com/brokeyourbike/go-mockery-action/releases)
[![codecov](https://codecov.io/gh/brokeyourbike/go-mockery-action/graph/badge.svg?token=hpF9JeJxc5)](https://codecov.io/gh/brokeyourbike/go-mockery-action)

Set up your GitHub Actions workflow with a specific version of [mockery](https://github.com/vektra/mockery)

## Usage

See [action.yml](action.yml)

Basic:
```yaml
steps:
  - uses: actions/checkout@v6
  - uses: brokeyourbike/go-mockery-action@v0
    with:
      mockery-version: '2.9.4' # The mockery version to download and use.
  - run: mockery --all
```

## Arguments

| Input  | Description | Usage |
| :---:     |     :---:   |    :---:   |
| `mockery-version`  | [mockery](https://github.com/vektra/mockery) version to download and use  | *Required |

## Authors

- [Ivan Stasiuk](https://github.com/brokeyourbike) | [Twitter](https://twitter.com/brokeyourbike) | [LinkedIn](https://www.linkedin.com/in/brokeyourbike) | [stasi.uk](https://stasi.uk)

## License

The scripts and documentation in this project are released under the [MPL-2.0](https://github.com/brokeyourbike/go-mockery-action/blob/main/LICENSE)

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
