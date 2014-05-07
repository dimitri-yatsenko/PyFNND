PyFNND
=======
A Python implementation of Fast Non-Negative Deconvolution
------------------------------------

This is a Python implementation of Joshua Vogelstein's fast non-negative deconvolution algorithm for inferring spikes from neuronal calcium imaging data. Fast non-negative deconvolution uses an interior point method to solve the following optimization problem:

![img]http://bit.ly/1qfgrbd[/img]

where n_best is the maximum a posteriori estimate for the most likely spike train, given the fluorescence signal F, and the model:

![img]http://bit.ly/1mDt22h[/img]

It is also possible to estimate the model parameters sigma, beta and lambda from the data using pseudo-EM updates.

This particular version was written for the [Kaggle Connectomics Challenge](https://www.kaggle.com/c/connectomics), and was optimized for speed when operating on large arrays of fluorescence data. In particular, it wraps the [`dgtsv`](http://www.netlib.org/lapack/explore-html/d1/db3/dgtsv_8f.html) subroutine from the LAPACK Fortran library in order to efficiently solve for the update direction for each Newton step. The optional dependency on [`joblib`](https://pythonhosted.org/joblib) allows multiple fluorescence traces to be processed in parallel. It is possible to obtain spike train estimates for 1000 neurons sampled at 50Hz for 1 hour each within about 10 min on a modern quad core laptop.

Dependencies
-------------
* `numpy` - must be built against a LAPACK shared library
* `scipy`
* `joblib` - optional, required for parallel processing

Example useage
-----------------

    from matplotlib import pyplot as plt
    import numpy as np
    from scipy import stats, ndimage
    from pyfnnd import fnndeconv, demo

    # generate some simulated spike trains, calcium and fluorescence traces for
    # 100 cells
    nc = 100
    nt = 10000
    S, C, F = demo.make_fake_data(100, nt, dt=0.02)

    # process all 100 cells simultaneously (requires joblib)
    N, C_hat, LL, Theta = fnndeconv.apply_all_cells(F, disp=False)

    # plot the fit for the first 5 cells
    fig, (ax1, ax2) = plt.subplots(2, 1, sharex=True)
    for aa in (ax1, ax2):
        aa.hold(True)
    t = np.arange(nt) * 0.02

    # real and fitted fluorescence
    real = ax1.plot(t, F[:5].T + np.arange(5) * 2, '-b')[0]
    fit = ax1.plot(t, C_hat[:5].T + Theta[:5, 1] + np.arange(5) * 2, '-r')[0]
    ax1.set_ylabel('Fluorescence')

    # real and inferred spike probabilities
    ax2.plot(t, S[:5].T + np.arange(5) * 1.1, '-b')
    ax2.plot(t, N[:5].T * 4 + np.arange(5) * 1.1, '-r')
    ax2.set_ylabel('Spikes')

    fig.legend((real, fit), ('Real', 'Inferred'), loc=9, ncol=2, fancybox=True)

    plt.show()

![Imgur](http://i.imgur.com/BxRRKA6.png)

Reference
----------
Vogelstein, J. T., Packer, A. M., Machado, T. a, Sippy, T., Babadi, B., Yuste, R., & Paninski, L. (2010). Fast nonnegative deconvolution for spike train inference from population calcium imaging. Journal of Neurophysiology, 104(6), 3691-704. doi:10.1152/jn.01073.2009
