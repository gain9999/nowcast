# TMD Nowcast Radar API (Client-Side)

A completely serverless, client-side API that extracts real-time and forecasted rain intensity data for specific locations in Bangkok. It directly processes radar imagery from the Thai Meteorological Department (TMD) to provide structured JSON data.

**Base URL:** [http://gain9999.github.io/nowcast](http://gain9999.github.io/nowcast)

---

## 📡 Data Sources & Acknowledgements

This API relies on publicly available weather data and imagery provided by government meteorological agencies in Thailand. We gratefully acknowledge the following sources:

*   **Primary Radar Source:** The raw radar data originates from the **Bangkok Nongchok Radar**, operated by the Bangkok Metropolitan Administration (BMA). You can view the official BMA radar animation at [weather.bangkok.go.th](https://weather.bangkok.go.th/Radar/RadarAnimation.aspx).
*   **Nowcast Processing:** The processed 3-hour Nowcast models and image overlays are generated and hosted by the **Thai Meteorological Department (TMD)**.
*   **Data Endpoints:** The image data is specifically sourced from the [TMD Satellite Data Services (SATDA)](http://satda.tmd.go.th/), specifically the [Bangkok Nowcasting visualizer](https://satda.tmd.go.th/wp-content/uploads/nowcasting/bkk/bangkok.php).

*Disclaimer: This API is a third-party tool designed to make the official data machine-readable. It is not officially affiliated with or endorsed by TMD or the BMA.*

---

## 📖 How to Use It

Because this API runs entirely in the browser (using JavaScript and the HTML5 Canvas API), you interact with it by pointing a web browser or a headless browser tool at the Base URL and passing your target location via GET parameters.

### URL Parameters

| Parameter | Type | Required? | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `lat` | Float | No* | `13.7563` | The latitude of your target location. |
| `lon` | Float | No* | `100.5018` | The longitude of your target location. |
| `radius` | Float | No | `3` | The radius (in kilometers) around the target location to analyze. |

*\*If `lat` and `lon` are not provided, the page will attempt to prompt the user for their browser's geolocation. If denied, it falls back to the default coordinates (Central Bangkok).*

### Example Request
```text
http://gain9999.github.io/nowcast?lat=13.7322&lon=100.5481&radius=3
```

---

## 📤 Output Format Explanation

Upon a successful request, the page will automatically process the radar data and replace its content with a raw, pretty-printed JSON response.

```json
{
  "metadata": {
    "lat": 13.7322,
    "lon": 100.5481,
    "radius_km": 5,
    "generated_at": "2026-05-09T14:45:00+07:00"
  },
  "forecasts": [
    {
      "timestamp": "2026-05-09T15:00:00+07:00",
      "status": "clear",
      "percent_rain": 0
    },
    {
      "timestamp": "2026-05-09T15:15:00+07:00",
      "status": "heavy",
      "percent_rain": 42.5
    }
  ]
}
```

### Key Definitions:
*   **`metadata.generated_at`**: The exact local Thailand time (`+07:00`) when the TMD server finished generating this batch of radar images.
*   **`forecasts.timestamp`**: The local Thailand time (`+07:00`) for the specific 15-minute forecast frame.
*   **`status`**: The **absolute maximum** rain intensity detected *anywhere* within your specified radius. This acts as a worst-case scenario alert. 
    *   *Values:* `"clear"`, `"drizzle"`, `"light"`, `"moderate"`, `"heavy"`, `"very_heavy"`.
*   **`percent_rain`**: The percentage (0 to 100) of the area inside your radius that is experiencing **"light" rain or heavier**. *(Note: "drizzle" is intentionally excluded from this calculation to filter out negligible rainfall).*

---

## ⚙️ The Logic Behind the Scenes

1.  **Dynamic Timeline Probing:** The script calculates the current time in Thailand and generates a timeline of 15-minute intervals. It aggressively probes the TMD server for up to 20 frames (covering current conditions and up to a 3-hour forecast).
2.  **Geographic Mapping:** It loads the transparent PNG radar overlays into memory. Using the official geographic bounding box coordinates provided by TMD, it maps every single pixel in the image to a real-world Latitude and Longitude.
3.  **Haversine Distance:** It calculates the distance of every pixel from your requested `lat`/`lon` using the Haversine formula (accounting for the Earth's curvature). If a pixel falls within your `radius`, it is analyzed.
4.  **Color Classification:** It extracts the `rgba` (Red, Green, Blue, Alpha) values of the active pixels. It compares these colors against the strict hex color codes used in the official TMD "Rain Criteria" legend to classify the intensity of the rain at that specific coordinate.

---

## ⚠️ Limitations and Uncertainty

When using this data, please keep the following limitations in mind:

*   **Spatial Resolution:** The raw TMD radar images have a resolution of approximately **1.1 km per pixel**. Requesting a very small radius (e.g., 1km) means you are only analyzing 1 to 4 pixels, which can lead to high variance.
*   **Radius Interpretation:** 
    *   A **large radius (e.g., 10km)** is excellent for early warnings, but a storm hitting the edge of your 10km circle will trigger a `"heavy"` status, even if your exact house remains dry. The `percent_rain` metric helps contextualize this.
    *   A **small radius (e.g., 2km)** accurately reflects your immediate surroundings but gives you less warning time for incoming weather.
*   **Update Frequency:** TMD generates these forecasts in batches. The data is not a live, second-by-second feed; it updates incrementally based on their internal modeling schedule.
*   **Coverage Area:** If you request coordinates outside the Greater Bangkok radar bounding box, the API will immediately return an error JSON to prevent processing empty data.

---

## 🚀 Practical Applications

Because the output is structured JSON, this API is highly versatile for automated systems:

*   **Smart Home Automation:** Trigger a routine to close automated windows or send a push notification to your phone if the `status` changes to `"heavy"` in your 3km radius.
*   **Delivery/Logistics Routing:** Delivery services can check the `percent_rain` along a specific route and warn drivers or adjust estimated delivery times based on impending heavy rain.
*   **Event Planning:** Outdoor venues can monitor the 3-hour forecast array to make go/no-go decisions for concerts or sports matches.
*   **Personal Dashboards:** Embed the data into a Magic Mirror or a local Grafana dashboard for a hyper-local weather display.