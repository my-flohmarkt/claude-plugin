# Public catalogue tool contract

Server: `https://mcp.my-flohmarkt.de/mcp`. MCP Streamable HTTP, configured as
`type: http` in Claude's `.mcp.json`. Authentication: none.

All three tools declare `readOnlyHint: true` and `destructiveHint: false`.
They operate on the public My Flohmarkt catalogue and do not expose write or
account operations.

## search_events

All arguments are optional:

| Argument             | Meaning and constraint                                      |
| -------------------- | ----------------------------------------------------------- |
| `query`              | Free search text; at most 200 characters.                   |
| `city`               | City name or PLZ string; at most 100 characters.            |
| `dateFrom`, `dateTo` | Date boundaries in `YYYY-MM-DD` format.                     |
| `lat`                | Latitude between -90 and 90.                                |
| `lng`                | Longitude between -180 and 180.                             |
| `radiusKm`           | Radius from 1 to 100 km. Supply all three radius arguments. |
| `limit`              | Integer from 1 to 50; server default 10.                    |

The tool returns `items`, `total`, `totalIsLowerBound` and, when available,
`dataAsOf`. An item has `id`, `title`, `date`, `plz`, `city`, `url` and available
summary fields such as `timeFrom`, `timeTo`, coordinates and market type.
Treat optional and null values as missing. `totalIsLowerBound: true` means the
count is not a complete total. There is no offset argument.

Example for an explicit request around Munich's Marienplatz:

```json
{
  "lat": 48.1372,
  "lng": 11.5756,
  "radiusKm": 25,
  "limit": 5
}
```

## get_event

Required argument: `id`, a public event's UUID. Return: `event` with available
public details, or `error: not_found`. Detail fields can include street and house
number, opening times, description, entrance fee, seller fee and the public
event URL. `entranceFee` and `sellerFee` describe different charges; do not
infer either from a missing value or from a generic `fee` field.

## recently_updated_events

Optional argument: `limit`, integer from 1 to 50, default 10. Returns `items`
in descending catalogue-change order and, when available, `dataAsOf`.
The event's `date` is distinct from its catalogue update time.

## Errors and freshness

Inputs are validated. A partial radius filter is invalid. A tool error, rate
limit or service outage is not evidence that no matching events exist. Respect
`Retry-After` when the transport supplies it; do not repeatedly retry.

Data comes from a periodically refreshed snapshot. Missing details and delayed
updates are possible. Links identify the returned public record; they do not
guarantee that an event will take place.

[Published connection documentation](https://my-flohmarkt.de/mcp.md)
