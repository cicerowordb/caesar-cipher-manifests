# Grafana configuration

* Versions used for this installation:
  * GoLang Version: `1.21.8`
  * Grafana version: `10.4.0`

### Setting Up Telegram for Alert Messaging

1. Access the Grafana WUI.
2. Go to `Home > Alerting > Contact Points`:
    1. Click the `+ Add contact point` button.
        1. Name: `Telegram Bot`
        1. Integration: `Telegram`
        1. Enter your `BOT API Token` (be careful).
        1. Enter your `Chat ID` (be careful).
        1. Click the `Test` button.
        1. Click the `Save contact point` button.
3. Go to `Home > Alerting > Notification policies`:
    1. Select the option `...` for **Default policy** and click `Edit`.
    1. Change the default contact point to `Telegram Bot`.
    1. Click `Update default policy`.
4. Go to `Home > Dashboards`:
    1. Click the `New` button and select `New folder`.
        1. Folder name: `alerts`
        1. Click `Create`.

### Setting Up an Alert

1. Apply the dashboard described in `grafana-dashboards/workload-dashboard-requests-error.json`.
2. Edit the first panel: `Incoming Requests By Source And Response Code`:
    1. Go to `Alert`.
    1. Click `New alert rule`.
        1. At `C - Threshold`, set `is above` to 1.
        1. At `Folder`, choose `alerts`.
        1. Click `+ New evaluation group`.
            1. Set name: `eval-group`.
            1. Set time: `2m`.
        1. Click the `Save rule and exit` button.