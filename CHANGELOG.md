# Changelog

## Qt6 Port (2026-06-11)

Ported the project from Qt4/Qt5 to Qt6.9.0 on macOS arm64 (Apple clang 17, Darwin 25.2).

### Build system (`kvant.pro`)

- Removed `CODECFORTR = Windows-1251` — not supported in Qt6.
- Added `CONFIG += sdk_no_version_check` to suppress macOS SDK version mismatch warnings.
- Added post-`qmake` Makefile patch to strip `-framework AGL`, which was removed in macOS 14 but is still referenced in Qt6's `macx-clang` mkspecs via `QMAKE_LIBS_OPENGL`.

### API removals — fixed across all affected files

#### `Qt::WindowFlags` default argument (`= 0` → `= Qt::WindowFlags()`)
Qt6 tightened `QFlags<T>` to reject implicit conversion from `int`.
Affected headers: `BoundaryCondition.h`, `LevelNumber.h`, `GView.h`, `Zview.h`, `ScaleWPX.h`, `ScalesUx.h`, `ScalePsin.h`, `MainWindow.h`, `WPparametersM.h`, `TimeView.h`, `PotentialViewMultiWell.h`, `Uparabolic.h`, `Uch2x.h`, `Ulinear.h`, `ProgramSetting.h`, `WPparametersP.h`, `ScaleTE.h`, `ScalePhin.h`, `ScaleWPK.h`, `plotter.h`, `PlotterDialog.h`.

#### `QString::sprintf()` → `QString::asprintf()` (static)
`QString::sprintf()` (instance method) was removed in Qt6. Replaced ~80 call sites of the form `buf.sprintf(fmt, ...)` with `buf = QString::asprintf(fmt, ...)`.
Affected files: `GView.cpp`, `LevelNumber.cpp`, `TimeView.cpp`, `Zview.cpp`, `ScaleWPK.cpp`, `ScaleWPX.cpp`, `ScalePhin.cpp`, `ScalePsin.cpp`, `ScalesUx.cpp`, `LineEditDelegate.cpp`, `PlotterDialog.cpp`, `EGView.cpp`, `EnzView.cpp`, `EofqaView.cpp`, `TransmissionView.cpp`, `TofzView.cpp`, `WaveFunctionView.cpp`, `WPparametersM.cpp`, `WPparametersP.cpp`, `ProgramSetting.cpp`, `PotentialViewMultiWell.cpp`, `Uch2x.cpp`, `Uparabolic.cpp`, `Ulinear.cpp`, `myparam.cpp`.

#### `QMatrix` → `QTransform`
`QMatrix` was removed in Qt6. Replaced include, type, and all associated method calls:
- `#include <QMatrix>` → `#include <QTransform>`
- `QMatrix m` → `QTransform m`
- `view->matrix()` → `view->transform()`
- `this->setMatrix(m)` → `this->setTransform(m)`
- Bare `matrix().scale(...)` → `transform().scale(...)`

Affected files: `EGView.cpp`, `EnzView.cpp`, `EofqaView.cpp`, `MomentumView.cpp`, `TransmissionView.cpp`, `TofzView.cpp`, `WaveFunctionView.cpp`, `WavePacketKView.cpp`, `WavePacketXView.cpp`, `PotentialViewMovable.cpp`.

#### `QWheelEvent::delta()` → `QWheelEvent::angleDelta().y()`
`QWheelEvent::delta()` was removed in Qt6.
Affected files: `EGView.cpp`, `EnzView.cpp`, `EofqaView.cpp`, `MomentumView.cpp`, `TransmissionView.cpp`, `TofzView.cpp`, `WaveFunctionView.cpp`, `WavePacketKView.cpp`, `WavePacketXView.cpp`, `PotentialViewMovable.cpp`, `Plotter.cpp`.

#### `QWheelEvent::orientation()` removed (`Plotter.cpp`)
Replaced `event->orientation() == Qt::Horizontal` with `event->angleDelta().x() != 0`.

#### `QRegion::rects()` removed (`Plotter.cpp:paintEvent`)
`QRegion::rects()` returning `QVector<QRect>` was removed in Qt6. Replaced the index-based loop with a range-for over `event->region()`.

#### `enterEvent(QEvent*)` → `enterEvent(QEnterEvent*)` (`Plotter.cpp`, `plotter.h`)
Qt6 changed the `enterEvent` virtual signature to use `QEnterEvent*`.

#### `QPixmap::fill(const QWidget*, int, int)` removed (`Plotter.cpp`)
Replaced `pixmap.fill(this, 0, 0)` with `pixmap.fill(palette().window().color())`.

#### `QString::null` removed (`Plotter.cpp`, `MainWindow.cpp`)
`QString::null` was removed in Qt6. Replaced with `QString()`.

#### `QRegExp` / `QRegExpValidator` removed (`MainWindow.cpp`)
Both were removed in Qt6. Replaced with `QRegularExpression` / `QRegularExpressionValidator` and `setRegularExpression()`. Added `#include <QRegularExpression>` and `#include <QRegularExpressionValidator>`.

### `main.cpp`
- Removed `#include <QTextCodec>` (class removed in Qt6; the actual usage was already commented out).
- Replaced `qsrand(QTime(0,0,0).secsTo(QTime::currentTime()))` with `QRandomGenerator::global()->seed(QTime::currentTime().msecsSinceStartOfDay())`.

### Runtime crash fix (`main.cpp`)
`QRandomGenerator::global()->seed()` is forbidden in Qt6 — the global RNG is OS-seeded and calling `seed()` on it triggers `qFatal()`. Removed the call entirely; the global RNG requires no manual seeding.

### Window title (`main.cpp`)
Replaced garbled Windows-1251 Russian title with English: `"Solution of the Schroedinger equation for a piecewise-constant potential"`.

## UI string translations (2026-06-11)

The source file encoding was Windows-1251; a prior bad UTF-8 conversion permanently replaced every Russian character with the Unicode replacement character (U+FFFD), making all Russian strings unrecoverable. All affected strings have been replaced with English equivalents.

### `MainWindow.cpp`

| Location | Old (garbled) | New (English) |
|---|---|---|
| Menu bar | `Potential U(x)` menu title (Russian) | `"Potential U(x)"` |
| Potential menu actions | Russian names | `"Step-wise"`, `"Linear"`, `"Parabolic"` |
| File menu | About action (Russian) | `"&About"` |
| Menu bar | Graphs menu title (Russian) | `"Graphs"` |
| Graphs menu | `wMenu->setWhatsThis` | `"Boundary conditions for the model"` |
| Graphs > U(x)&&En | `setStatusTip` | `"Graph of potential and levels U(x)"` |
| Graphs > ψ_n(x) | `setStatusTip` | `"Wave functions for energy levels"` |
| Graphs > \|φ_n(k)\|² | `setStatusTip` | `"Momentum distributions for energy levels"` |
| Graphs > Ψ(x,t) | `setStatusTip` | `"Coordinate distribution: time development of wave packet"` |
| Graphs > \|Φ_n(k,t)\|² | `setStatusTip` | `"Momentum distribution: time development of wave packet"` |
| Graphs > T(E) | `setStatusTip` | `"Transmission coefficient vs energy"` |
| Menu bar | Additional menu title (Russian) | `"Additional"` |
| Additional > T(z) | `setStatusTip` | `"Transmission coefficient vs z"` |
| Additional > Enz | `setStatusTip` | `"Energy levels vs parameter z"` |
| Additional > qa(E) | `setStatusTip` | `"Quasi-levels in complex potential"` |
| Additional > B_{N+1}(E+iG) | `setStatusTip` | `"Resonance levels E+iG, Re and Im B_{N+1}"` |
| Toolbar: Model | `setToolTip` | `"Boundary conditions"` |
| Toolbar: Un | `setToolTip` | `"Multi-well/barrier potential"` |
| Toolbar: En | `setToolTip` | `"Values of energy levels En"` |
| Toolbar: (E+iG)n | `setToolTip` | `"Values of quasi-energy levels"` |
| Toolbar: U1 | `setToolTip` | `"Initial potential"` |
| Toolbar: U2 | `setToolTip` | `"Final potential"` |
| Toolbar: Ubias | `setToolTip` | `"Applied bias voltage"` |
| Toolbar: Settings | `setToolTip` | `"Line width in pixels"` |
| About dialog | Russian HTML body | English text with authors and project URL |

### `PotentialViewMovable.cpp`
- `setWhatsThis` → `"In this window you can drag potential wells and barriers with the mouse."`
