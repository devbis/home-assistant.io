---
title: Google Cast
description: Instructions on how to integrate Google Cast into Home Assistant.
ha_category:
  - Media Player
featured: true
ha_release: pre 0.7
ha_iot_class: Local Polling
ha_config_flow: true
ha_domain: cast
ha_codeowners:
  - '@emontnemery'
---

You can enable the Cast integration by going to the Integrations page inside the configuration panel.

## Setup

Support for mDNS discovery in your local network is mandatory. Make sure that your router has this feature enabled. This is even required if you entered the IP addresses of the Cast devices are manually in the configuration as mentioned below.

## Home Assistant Cast

Home Assistant has its own Cast application to show the Home Assistant UI on any Chromecast device.  You can use it by adding the [Cast entity row](/lovelace/entities/#cast) to your Lovelace UI, or by calling the `cast.show_lovelace_view` service. The service takes the path of a Lovelace view and an entity ID of a Cast device to show the view on. A `path` has to be defined in your Lovelace YAML for each view, as outlined in the [views documentation](/lovelace/views/#path). The `dashboard_path` is the part of the Lovelace UI URL that follows the defined `base_url` Typically "lovelace". The following is a full configuration for a script that starts casting the `downstairs` tab of the `lovelace-cast` path (note that `entity_id` is specified under `data` and not for the service call):

```yaml
'cast_downstairs_on_kitchen':
  alias: Show Downstairs on kitchen
  sequence:
  - data:
      dashboard_path: lovelace
      entity_id: media_player.kitchen
      view_path: downstairs
    service: cast.show_lovelace_view
```
<div class='note'>

Home Assistant Cast requires your Home Assistant installation to be accessible via `https://`. If you're using Home Assistant Cloud, you don't need to do anything. Otherwise you must make sure that you have configured the `external_url` in your [configuration](/docs/configuration/basic).

</div>

## Casting other apps

### YouTube

- `app_name`: `youtube`
- `media_id`: YouTube video ID

Optional:
- `enqueue`: Enqueue only
- `playlist_id`: Play video with `media_id` from this playlist

```yaml
'cast_youtube_to_my_chromecast':
  alias: Cast YouTube to My Chromecast
  sequence:
  - data:
      entity_id: media_player.my_chromecast
      media_content_type: cast
      media_content_id: '
        {
          "app_name": "youtube",
          "media_id": "dQw4w9WgXcQ"
        }'
    service: media_player.play_media
```

### [Supla](https://www.supla.fi/)

Example values to cast the item at https://www.supla.fi/audio/3601824

- `app_name`: `supla`
- `media_id`: Supla item ID

Optional:
- `is_live`: Item is a livestream

```yaml
'cast_supla_to_my_chromecast':
  alias: Cast supla to My Chromecast
  sequence:
  - data:
      entity_id: media_player.my_chromecast
      media_content_type: cast
      media_content_id: '
        {
          "app_name": "supla",
          "media_id": "3601824"
        }'
    service: media_player.play_media
```

## Advanced use

### Manual configuration

By default, any discovered Cast device is added to Home Assistant. This can be restricted by supplying a list of allowed chrome casts.

```yaml
# Example configuration.yaml entry
cast:
  media_player:
    - uuid: "ae3be716-b011-4b88-a75d-21478f4f0822"
```

{% configuration %}
media_player:
  description: A list that contains advanced configuration options.
  required: false
  type: list
  keys:
    uuid:
      description: UUID of a Cast device to add to Home Assistant. Use only if you don't want to add all available devices. The device won't be added until discovered through mDNS.
      required: false
      type: string
    ignore_cec:
      description: >
        A list of Chromecasts that should ignore CEC data for determining the
        active input. [See the upstream documentation for more information.](https://github.com/balloob/pychromecast#ignoring-cec-data)
      required: false
      type: list
{% endconfiguration %}

### Docker and Cast devices and Home Assistant on different subnets

Cast devices can only be discovered and connected to if they are on the same subnet as Home Assistant.

When running Home Assistant Core in a [Docker container](/docs/installation/docker/), the command line option `--net=host` or the compose file equivalent `network_mode: host` must be used to put it on the host's network, otherwise the Home Assistant Core will not be able to connect to any Cast device.

Setups with cast devices on a different subnet than Home Assistant are not recommended and not supported.

If this is not possible, it's necessary to:

- Enable mDNS forwarding between the subnets.
- Enable source NAT to make requests from Home Assistant to the Chromecast appear to come from the same subnet as the Chromecast.
