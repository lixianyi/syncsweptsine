# Synchronized Swept Sine Method

This project implements the Synchronized Swept Sine Method as a reusable python package.
It is structured according to the paper by Nowak et al. 2015, but equations and symbol names are adapted to code conventions, also known as PEP 8.
Anyway, references to symbols and equations are given in the code comments. 
Most important classes are

- `SyncSweep` for the generation of the synchronized swept sine singal
- `HigherHarmonicImpulseResponse` for the deconvolution from sweep input and output signal.
- `HammersteinModel` filtering of signals with the hammerstein model.
- `LinearModel` filtering of signals with the linear kernel e.g.  from a HigherHarmonicImpulseResponse

Examples are placed in the examples folder.


```python
import numpy as np
from syncsweptsine import SyncSweep
from syncsweptsine import HigherHarmonicImpulseResponse
from syncsweptsine import HammersteinModel

sweep = SyncSweep(startfreq=16, stopfreq=16000, durationappr=10, samplerate=96000)

def nonlinear_system(sig):
    return 1.0 * sig + 0.25 * sig**2 + 0.125 * sig**3

outsweep = nonlinear_system(np.array(sweep))
hhir = HigherHarmonicImpulseResponse.from_sweeps(sweep, outsweep)
hm = HammersteinModel.from_higher_harmonic_impulse_response(
    hhir, 2048, orders=(1, 2, 3), systemdelay=0)
for kernel, order in zip(hm.kernels, hm.orders):
    print('Coefficient estimate of nonlinear system:', 
            np.round(np.percentile(abs(kernel.frf), 95), 3), 
            'Order', 
            order)
```
prints out:

```
Coefficient estimate of nonlinear system: 1.009 Order 1
Coefficient estimate of nonlinear system: 0.25 Order 2
Coefficient estimate of nonlinear system: 0.125 Order 3
``` 
            