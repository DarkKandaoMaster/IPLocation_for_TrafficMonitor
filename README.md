# IPLocationPlugin (TrafficMonitor Plugin)

[中文说明](README_zh-CN.md)

## Preview

![Preview](https://picgo-lenblog.oss-cn-beijing.aliyuncs.com/lenblog/202603161907585.png)

## Accurate IPLocation Plugin for TrafficMonitor | Real-time IP & Region
### Description
This plugin displays the public IPv4 address reported by external lookup services, with geographic information in the tooltip. It polls automatically; the timing and selection rules are described below.
### Useful scenarios
Check VPN/proxy node switching
Check public IP quickly for remote development & server configuration
Troubleshoot network issues with real egress IP
Monitor IP status across multiple network environments

TrafficMonitor is a lightweight Windows tool. This plugin adds one feature that is especially useful if you work with VPN/proxy nodes:

- Show the **current public (egress) IP** on the taskbar window
- Provide a tooltip for quick copy/verification

After switching nodes, the next completed poll updates the displayed address. With proxy routing rules, different lookup services may observe different egress IPs.

- Address lookup: `ipv4.icanhazip.com`, `api4.ipify.org`, `v4.ident.me`, `ipinfo.io/ip`, `api.ip.sb/ip`
  - Three are queried **simultaneously** per poll. The batch advances by one source
    each time, and the poll waits for all three requests to finish.
  - The most frequent valid IPv4 wins; one valid response is sufficient. Ties prefer
    the last published IP if it is among the tied results; otherwise the first tied
    result in batch order wins. A failed poll clears this preference.
  - JSON fallback, only if all three failed: `ip-api.com`, `ipwho.is`
- Location lookup (tooltip only): `ipwho.is` (`lang=zh-CN`), falling back to `ip-api.com`.
  Only the last successful location is cached. A matching IP reuses it; otherwise a
  lookup runs after publishing the IP. Failed location lookups are retried on later polls.

Every request is sent with no-cache headers, `WINHTTP_FLAG_REFRESH` and a varying
`_=<tick>` parameter to reduce stale responses from proxy or CDN caches.

This plugin also provides tooltip text via `ITMPlugin::GetTooltipInfo()`.

## Files

- `PluginInterface.h`: Official TrafficMonitor plugin interface header (from TrafficMonitorPlugins repo)
- `IPLocationPlugin.*`: Plugin implementation (`ITMPlugin`)
- `IPLocationItem.*`: Display item implementation (`IPluginItem`)
- `NetworkHelper.*`: WinHTTP GET helper
- `HtmlParser.*`: Legacy HTML parser (kept for compatibility; currently not the primary path)
- `CMakeLists.txt`: Build script

## Compatibility

- TrafficMonitor: tested with `v1.85.1` (x64)
- Windows: Windows 10/11
- Architecture: the plugin DLL must match TrafficMonitor (x64 vs x86)

## Build (Windows)

### Requirements

- CMake
- MSVC toolchain (Visual Studio / Build Tools)

Notes:

- When using MSVC, build in a Developer shell (or run `VsDevCmd.bat`) so `cl.exe` / `nmake.exe` are available.
- For single-config generators (NMake/Ninja), use `-DCMAKE_BUILD_TYPE=Release`.

### Build with MSVC + NMake (recommended for your current setup)

Open the Visual Studio Developer environment (so `cl.exe` and `nmake.exe` are available), then run:

```powershell
cd D:\Project\Test\IP\IPLocationPlugin

# Clean old build cache (recommended when changing generators/options)
rmdir build -Recurse -Force

cmake -S . -B build -G "NMake Makefiles" -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

The output DLL is typically:

- `build\IPLocationPlugin.dll`

### Build with Visual Studio generator

If CMake can detect your Visual Studio installation:

```powershell
cd D:\Project\Test\IP\IPLocationPlugin
rmdir build -Recurse -Force

cmake -S . -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
```

## Install

1. Copy `IPLocationPlugin.dll` into TrafficMonitor's `plugins` folder.
   Example: `D:\Utils\TrafficMonitor\plugins\`
2. Restart TrafficMonitor.
3. Enable the item in:
   - Taskbar window context menu -> Display Settings
   - Or: More Functions -> Plugin Management

Tip: if you update the DLL, fully exit TrafficMonitor (ensure the process ends) before replacing the file.

## Behavior

- Auto refresh: waits **30 seconds after a successful update completes**, or **10 seconds after IP lookup fails**, before the next poll. Request time and any location lookup add to the interval between IP updates.
- Manual click refresh: not used (disabled)
- Taskbar item: the IPv4 address on its own; `Failed` when it could not be fetched
- Tooltip: shows the IP, one `地区` line (city preferred, then region, then country),
  `IP地址更新时间` (result publication time, also updated when location lookup succeeds) and the selected source's vote count, or the JSON fallback source

IPv4 only:

- Only valid IPv4 responses are accepted; an IPv6 response is ignored.

## Data quality notes

Location databases may disagree for proxy/datacenter IPs. This plugin prioritizes `ipwho.is` because it provides stable JSON access without requiring a browser.

If you see different country/region in another software, it is usually due to:

- Different geolocation database vendors
- IP ownership/route changes (ASN prefix moved)
- Proxy/VPN/datacenter tagging differences

## Troubleshooting

- If TrafficMonitor crashes on startup after copying the DLL, it is almost always an ABI mismatch.
  Ensure you are using the official `PluginInterface.h` that matches your TrafficMonitor version.
- If location looks inconsistent, note that different databases may return different results for proxy/datacenter IPs.

## License

MIT License. See `LICENSE`.

## Credits

- TrafficMonitor plugin system and interface header are provided by the TrafficMonitor project.

## Contact

- Email: 1416679017@qq.com
