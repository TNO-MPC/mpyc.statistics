# TNO MPC Lab - MPyC - Statistics

The TNO MPC lab consists of generic software components, procedures, and functionalities developed and maintained on a regular basis to facilitate and aid in the development of MPC solutions. The lab is a cross-project initiative allowing us to integrate and reuse previously developed MPC functionalities to boost the development of new protocols and solutions.

The package tno.mpc.mpyc.statistics is part of the TNO Python Toolbox.

Within the [LANCELOT](https://www.tno.nl/en/about-tno/news/2021/11/lancelot-new-collaboration-between-iknl-and-tno-to-enable-privacy-preserving-analyses-on-cancer-related-data/) project, a collaboration between TNO, IKNL and Janssen, TNO developed and implemented secure statistics. LANCELOT is partly funded by PPS-surcharge for Research and Innovation of the Dutch Ministry of Economic Affairs and Climate Policy. The Appl.AI project [SELECTED](https://nlaic.com/en/use-case/selected-using-data-to-provide-individualised-healthcare-with-attention-to-privacy-and-security-aspects/), partly funded by NLAIC, also contributed to specific components in secure statistics (correlation, covariance).

*Limitations in (end-)use: the content of this software package may solely be used for applications that comply with international export control laws.*  
*This implementation of cryptographic software has not been audited. Use at your own risk.*

## Documentation

Documentation of the tno.mpc.mpyc.statistics package can be found [here](https://docs.mpc.tno.nl/mpyc/statistics/0.1.1).

## Install

Easily install the tno.mpc.mpyc.statistics package using pip:
```console
$ python -m pip install tno.mpc.mpyc.statistics
```

If you wish to run the tests you can use:
```console
$ python -m pip install 'tno.mpc.mpyc.statistics[tests]'
```

### Note:
A significant performance improvement can be achieved by installing the GMPY2 library.
```console
$ python -m pip install 'tno.mpc.mpyc.statistics[gmpy]'
```

## Usage

The statistics module can be used as follows:

```python
import numpy as np
from mpyc.runtime import mpc
from tno.mpc.mpyc.statistics import covariance


secnum = mpc.SecFxp(l=64, f=32)


def get_mpc_data(row_1, row_2):
    row_1_mpc = [secnum(x) for x in row_1]
    row_2_mpc = [secnum(y) for y in row_2]
    return row_1_mpc, row_2_mpc


def distribute_data_over_players(row_1_mpc, row_2_mpc):
    row_1_mpc_shared = mpc.input(row_1_mpc, senders=0)
    row_2_mpc_shared = mpc.input(row_2_mpc, senders=0)
    return row_1_mpc_shared, row_2_mpc_shared


async def covariance_example():
    print("Covariance example")

    row_1 = [1.0, 3.0, 2.0, 1.0, 5.0, 6.0, 3.0]
    row_2 = [2.0, 11.0, 9.0, 0.0, 8.0, 2.0, 2.1]

    row_1_np = np.array(row_1)
    row_2_np = np.array(row_2)

    row_1_mpc, row_2_mpc = get_mpc_data(row_1_np, row_2_np)

    async with mpc:
        row_1_mpc_shared, row_2_mpc_shared = distribute_data_over_players(
            row_1_mpc, row_2_mpc
        )

    secure_cov = covariance(row_1_mpc_shared, row_2_mpc_shared)
    revealed_cov = await mpc.output(secure_cov)

    np_cov = np.cov(row_1, row_2)[0][1]

    print("Secure Covariance: ", revealed_cov)
    print("Numpy Covariance:", np_cov)


if __name__ == "__main__":
    mpc.run(covariance_example())
```
