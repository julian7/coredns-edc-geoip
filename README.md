# edcgeoip

edcgeoip is a copy of [coredns](https://coredns.io/)' geoip internal plugin, with EDC (EDNS Client Subnet) support.

It can be used as a drop-in replacement of the original `geoip` plugin.

## Build

* get [CoreDNS](https://github.com/coredns/coredns) repository
* add edcgeoip plugin URL to `plugin.cfg` in CoreDNS repo (or replace `geoip` with `edcgeoip:github.com/julian7/coredns-edc-geoip`)
* rebuild CoreDNS (run `make` in the repo)
* take the executable (`coredns` or `coredns.exe` depending on your environment)

Example:

```shell
$ git clone https://github.com/coredns/coredns
Cloning into 'coredns'...
...
$ cd coredns
$ sed -p -i"" -e "s|^geoip$|edcgeoip:github.com/julian7/coredns-edc-geoip|" plugin.cfg
$ make
```

## Name

*edcgeoip* - Lookup maxmind geoip2 databases using the client IP, then add associated geoip data to the context request.

## Description

The *edcgeoip* plugin add geo location data associated with the client IP, it allows you to configure a [geoIP2 maxmind database](https://dev.maxmind.com/geoip/docs/databases) to add the geo location data associated with the IP address.

The data is added leveraging the *metadata* plugin, values can then be retrieved using it as well, for example:

```go
import (
    "strconv"
    "github.com/coredns/coredns/plugin/metadata"
)
// ...
if getLongitude := metadata.ValueFunc(ctx, "geoip/longitude"); getLongitude != nil {
    if longitude, err := strconv.ParseFloat(getLongitude(), 64); err == nil {
        // Do something useful with longitude.
    }
} else {
    // The metadata label geoip/longitude for some reason, was not set.
}
// ...
```

## Databases

The supported databases use city schema such as `City` and `Enterprise`. Other databases types with different schemas are not supported yet.

You can download a [free and public City database](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data).

## Syntax

```text
edcgeoip [DBFILE]
```

or

```text
edcgeoip [DBFILE] {
    [enable-edc]
}
```

* **DBFILE** the mmdb database file path.
* `enable-edc`: Optional. Use [EDNS0 subnet](https://en.wikipedia.org/wiki/EDNS_Client_Subnet) (if present) for Geo IP instead of the source IP of the DNS request. This helps identifying the closest source IP address through intermediary DNS resolvers, and it also makes GeoIP testing easy: `dig +subnet=1.2.3.4 @dns-server.example.com www.geo-aware.com`.

## Examples

The following configuration configures the `City` database, and looks up geolocation based on EDNS0 subnet if present.

```txt
. {
    edcgeoip /opt/geoip2/db/GeoLite2-City.mmdb {
      enable-edc
    }
    metadata # Note that metadata plugin must be enabled as well.
}
```

## Metadata Labels

A limited set of fields will be exported as labels, all values are stored using strings **regardless of their underlying value type**, and therefore you may have to convert it back to its original type, note that numeric values are always represented in base 10.

| Label                                | Type      | Example          | Description
| :----------------------------------- | :-------- | :--------------  | :------------------
| `geoip/city/name`                    | `string`  | `Cambridge`      | Then city name in English language.
| `geoip/country/code`                 | `string`  | `GB`             | Country [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1) code.
| `geoip/country/name`                 | `string`  | `United Kingdom` | The country name in English language.
| `geoip/country/is_in_european_union` | `bool`    | `false`          | Either `true` or `false`.
| `geoip/continent/code`               | `string`  | `EU`             | See [Continent codes](#ContinentCodes).
| `geoip/continent/name`               | `string`  | `Europe`         | The continent name in English language.
| `geoip/latitude`                     | `float64` | `52.2242`        | Base 10, max available precision.
| `geoip/longitude`                    | `float64` | `0.1315`         | Base 10, max available precision.
| `geoip/timezone`                     | `string`  | `Europe/London`  | The timezone.
| `geoip/postalcode`                   | `string`  | `CB4`            | The postal code.

## Continent Codes

| Value | Continent (EN) |
| :---- | :------------- |
| AF    | Africa         |
| AN    | Antarctica     |
| AS    | Asia           |
| EU    | Europe         |
| NA    | North America  |
| OC    | Oceania        |
| SA    | South America  |
