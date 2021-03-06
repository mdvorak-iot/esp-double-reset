# esp-double-reset

![platformio build](https://github.com/mdvorak-iot/esp-double-reset/workflows/platformio%20build/badge.svg)

Detect double reset, which can be used to place program in special reconfiguration mode, like entering WiFi credentials.

<table>
	<tr><th>Platform</th><td>ESP-IDF</td></tr>
	<tr><th>Toolchain</th><td>idf / platformio</td></tr>
	<tr><th>Storage</th><td>NVS</td></tr>
	<tr><th>Dependencies</th><td>None</td></tr>
</table>

## Usage

You can either use this as standard git submodule, or reference it via `platformio.ini`:

```ini
[env]
lib_deps =
    https://github.com/mdvorak-iot/esp-double-reset.git#v1.0.0
```

Note that when used as platformio library, `Kconfig` is not available. Either add [Kconfig](./Kconfig) contents to 
your `Kconfig.projbuild`, or set `DOUBLE_RESET_DEFAULT_TIMEOUT` manually, via

```ini
[env]
build_flags = -D DOUBLE_RESET_DEFAULT_TIMEOUT=3000
```

### Example

```c
#include <double_reset.h>

void app_main() 
{
	// Initialize NVS
	esp_err_t ret = nvs_flash_init();
	if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND)
	{
		ESP_ERROR_CHECK(nvs_flash_erase());
		ret = nvs_flash_init();
	}
	ESP_ERROR_CHECK(ret);

	// Check double reset
	// NOTE this should be called as soon as possible, ideally right after nvs init
	bool reconfigure = false;
	ESP_ERROR_CHECK(double_reset_start(&reconfigure, 10000));

	if (reconfigure)
	{
		ESP_LOGI(TAG, "double reset detected!");
	}

	// Setup complete
	ESP_LOGI(TAG, "started");
}
```

For working example, see [example/main.cpp](example/main.cpp). To build it, run

```bash
platformio run -d example/
```
