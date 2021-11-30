..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   AuxTel pointing determination and CWFS measurements show non-repeatability and high-order dependencies.
   This technote describes the actions taken to diagnose the issue and implement a fix.


Observed Behavior
=================

.. _pointing_technote: https://tstn-014.lsst.io/v/DM-30308/runs/run-202106.html

- Struggled to obtain reliable and repeatable pointing of the Auxiliary Telescope

- Believed early-on that it was mostly due to inaccuracies in measurements and due to the M1 and M2 LUTs not be sufficiently stable due to a poor sampling of the observing range and/or poor fits to that data

- Significant time has been spent to accumulate more data for the pointing model, details are found in the pointing_technote_.
  The main conclusions are two-fold.

    #. There is a high-order "wiggle" around 30-40 degrees in elevation that is not well fit.
       This can be seen in the central plot of the following figure.
       It is also present in the center-left plot but is less obvious.
       The pointing component is better suited to smooth behaviors, so which this is not.

    #. There is significant scatter in the pointing observations, even for the same telescope position.
       Values can be up to 30 arcsec. This indicates a non-repeatable component in one of the mirror positions.


.. figure:: _static/tpoint_a9plot_ia_ie_an_aw_tf_tx10_20210608_tw004.png
    :name: tpoint_a9plot_ia_ie_an_aw_tf_tx10_20210608_tw004.png
    :target: _static/tpoint_a9plot_ia_ie_an_aw_tf_tx10_20210608_tw004.png
    :alt: pointing model fit

    The "9 favorite plots for alt-azimuth".
    This plot gathers the 9 most relevant plots with information about the pointing model fit for an altaz mount.
    **Top-left-hand:** The east-west residuals plotted against hour angle.
    **Top-center:** The declination residuals plotted against declination.
    **Top-right-hand:** Zenith-distance errors against zenith distance.
    **Center-left-hand:** The east-west residuals plotted against zenith distance.
    **Center:** The declination residuals plotted against hour angle.
    **Center-right-hand:** The residuals are interpreted as changes in the h/δ non-perpendicularity and plotted against hour angle.


- Significant time has also been spent accumulating data to create an M2 position look-up table.
  This is completed by performing CWFS measurements at multiple elevations.
  Again, there is scatter in the data that corresponds to about 0.2 mm of hexapod motion, which is equivalent to ~30 nm of WFE.

.. figure:: /_static/hexapod_combined.jpg
    :name: fig-hexapod_combined
    :target: ../_static/hexapod_combined.jpg
    :alt: hexapod_original_data

    The accumulation of multiple datasets where we measured the hexapod position as a function of elevation.

- Until recently, there has been no other external measurement to help constrain the problem.

- We currently have no temperature sensors, however, there have been nights where the temperature remains stable and these effects are still seen.
  For the data discussed below the temperature increased ~8 degrees C but then held stable for ~6 hours which is when we took the data.

- Hexapod corrections only include translation and piston (focus).
  No tilt correction is currently applied, up to now, it has been assumed that this is best is taken out by the pointing model.
  The tilt is currently set to what was measured by Roberto during optical alignment


Information from the Optical Model
==================================

- During the last run a dataset was taken both translating and tip/tilting the hexapod.

- For each datapoint, the hexapod was moved by a set amount, and CWFS datasets were taken to measure the induced aberration(s).

- These were then compared against the optical model.
  For this analysis, the most important information is as follows:

    #. For M1, a 1mm lateral shift of the mirror results in ~65 arcseconds of image motion.
       Confirming this experimentally is really hard as we must adjust the belly-band manually and therefore we have not done it.

    #. For M2, a 1mm displacement results in 54.7 arcsec of image motion.
       Experimentally we measure ~52.5 and ~50.5 in X and Y.

    #. For M2, 0.0022 degrees (8 arcsec) of tilt results in 30 arcsec of image motion
       For the 266mm diameter mirror this corresponds to 0.33 mm of (roughly) vertical motion.
       This has not been verified experimentally, although the data exists to do so.

- We have experimentally measured the WFE induced by tipping/tilting M2, and by translating.
  The following figure shows the effects of translation of the hexapod X-axis, which is aligned with the gravity vector

.. figure:: /_static/hexXoffset_sensitivity.jpg
    :name: fig-hexapod_X_sensitivity
    :target: ../_static/hexXoffset_sensitivity.jpg
    :alt: fig-hexapod_X_sensitivity

    The WFE induced by translating the AT M2 hexapod in the X-axis.
    The Y-axis is the same in magnitude but the axes of the aberrations are flipped.
    Note that previous datasets have yielded ~30% higher slopes, so note that there may be uncertainty here.

- The next figure shows the effect of tipping (rotation in U, which is about the Y-axis).
  Rotation about the X-axis is the same magnitude but the aberrations are seen in the other axis (e.g. Coma-X instead of Coma-Y)

.. figure:: /_static/hexUoffset_sensitivity.jpg
    :name: fig-hexapod_U_sensitivity
    :target: ../_static/hexUoffset_sensitivity.jpg
    :alt: fig-hexapod_U_sensitivity

    The WFE induced by tipping the AT M2 hexapod in the U-axis.
    The V-axis is the same in magnitude but the axes of the aberrations are flipped.

Efforts to Isolate the Issue(s)
===============================

- Decided to spend time investigating both image motion and hexapod repeatability using a single concentrated dataset

- For this run, we have installed 3 micrometers on the M1 mirror to measure translation in all three axes.

.. figure:: /_static/mitutoyo_placements.png
    :name: fig-mitutoyo_placements
    :target: ../_static/mitutoyo_placements.png
    :alt: fig-mitutoyo_placements

    Mitutoyo dial indicators have been positioned to in these locations to measure primary mirror motion as a function of elevation.

- A dedicated dataset was taken moving up and down in elevation continually taking CWFS datasets to look at hexapod motion
  An in-focus image was also taken to observe the repeatability in the pointing

- Based on impromptu testing and some back of the envelope calculations, it was hypothesized that the top end was possibly loose.
  We currently lack instrumentation to diagnose this effectively, but people are trying to secure cell phones to the spiders and tap them to measure the fundamental frequency, essentially to determine if any are loose. This is on-going.

- Initial tests have shown excessive vibration in the spider vane (turnbuckle) in the following image.
  When tapped, this strut would vibrate for 1-2s, whereas the others had no discernible vibration

.. figure:: /_static/Struts.jpg
    :name: fig-Struts
    :target: ../_static/Struts.jpg
    :alt: fig-Struts

    The indicated vane appears to show increased vibration relative to the others.

Mirror Motion from dataset on the night of July 6, 2021
=======================================================

- The following data have been collected by moving up and down in elevation and performing CWFS measurements and subsequently applying the corrections to minimize WFE.
  This basically collimates and focuses the telescope at each position.

- The following plot shows the M1 mirror and M2 hexapod position as a function of elevation after a few dips

.. figure:: /_static/mirror_motion_on_axes_parallel_to_gravity.jpg
    :name: fig-mirror_motion_on_axes_parallel_to_gravity
    :target: ../_static/mirror_motion_on_axes_parallel_to_gravity.jpg
    :alt: fig-mirror_motion_on_axes_parallel_to_gravity

    Mirror motion along the elevation axis.
    The M1 motions (left) show a smooth hysteresis curve with a separation of ~0.05 mm.
    The M2 motions show a ~0.2mm RMS scatter, primarily at high altitude.
    The M2 motion is also not a low-order smoothly varying function.

- The following plot shows the M1 mirror and M2 hexapod position as a function of elevation for the same dataset, but now showing the axis perpendicular to elevation (and not aligned with gravity).

.. figure:: /_static/mirror_motion_on_axes_perpendicular_to_gravity.jpg
    :name: fig-mirror_motion_on_axes_perpendicular_to_gravity
    :target: ../_static/mirror_motion_on_axes_perpendicular_to_gravity.jpg
    :alt: fig-mirror_motion_on_axes_perpendicular_to_gravity

    Mirror motion perpendicular to the elevation axis (and gravity vector)
    The M1 motions (left) show a smooth but reduced hysteresis curve with a separation of ~0.03 mm.
    The M2 motions show a ~0.2mm RMS scatter; the elevation dependence is less clear.
    Again, the M2 motion is also not a smoothly varying function.


Motion Summary
--------------

Based on the observed motions we can use the sensitivity matrices to calculate their contributions

- M1 motion of 0.05 mm is the same as displacing M2, therefore assuming a worst case scenario this would result in 139*0.05 = 7 nm of WFE without correction and ~2.6 arcsec of image motion.
  In reality because this is a smooth function it is probably lower than this.
  It is believed that M1 lateral displacement cannot be the source of the large non-repeatability.

- M2 "random" motion in each axis of 0.2 mm RMS results in a WFE of ~28 nm.
  This also results in ~10 arcsec of pointing error per axis
  The PSF motion (pointing error) from this dataset has not yet been completed, however, based on the pointing model deviations of ~30 arcsec, it is anticipated that a tip/tilt motion is also required to account for this.

- If we assume 20 arcsec of pointing error comes from tip/tilt, this would result in only 0.9 nm of WFE, which we'd never be able to measure

- Based on the vibration data, a loose top-end seems consistent, but is very inconclusive

- The spec for the telescope pointing requirement is 10 arcsec over the entire sky.
  The non-repeatability needs to be addressed to meet this requirement.
  M1 lateral motion will eventually limit pointing accuracy but is not currently a significant contribution to the error budget.



Conclusions based on data to date
----------------------------------

#. A single measurement device on the back of M1 is not sufficient.
   It is possible that the mirror is tipping/tilting and we can't sense it.
   This could be from belly band slippage and therefore applying a lateral torque or from the mirror being lifted and moving.

#. The top end needs to be torqued.

Daytime work of July 6, 2021
============================

More mitutoyo sensors added, now have 5 sensors to get full M1 mirror motion measurements (except rotation).

.. figure:: /_static/mitutoyo_placements2.png
    :name: fig-mitutoyo_placements2
    :target: ../_static/mitutoyo_placements2.png
    :alt: fig-mitutoyo_placements2

The physical positioning of the sensors that is used in fitting the place representing the mirror is shown `_static </_static/>`_ directory having the filename(s) of "mirror_sensor_position_measurementX.jpg"

CWFS repeatability from dataset taken July 7, 2021
==================================================

.. _July7CWFSNotebook: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_20210607/CWFS_repeatability_Test-2021-07-07.ipynb?_xsrf=2%7C30ffc635%7C117a70f51a2cb57c083aec021346b1d6%7C1625867799

Reduced using July7CWFSNotebook_.

Took 10 CWFS sequences, correcting after each sequence, at a target near the pole where there was essentially no motion.

Following figure proves no telescope motion during the dataset

.. figure:: /_static/CWFS_repeatability_telescope_position.jpg
    :name: fig-CWFS_repeatability_telescope_position.jpg
    :target: ../_static/CWFS_repeatability_telescope_position.jpg
    :alt: fig-CWFS_repeatability_telescope_position.jpg

    The amount of telescope and hexapod motions measured for 10 consecutive sequences.
    The telescope basically remains stable.
    The hexapod motion is due to corrections from the CWFS iterations.

The zernike measurements represents the amount of "noise" whenever CWFS data is taken.
Note that this is a single instance, therefore if there is a dependence upon dome or mirror seeing then this may be an inaccurate measurements.
This is especially true if the seeing components are small.

.. figure:: /_static/CWFS_repeatability_zernikes.jpg
    :name: fig-CWFS_repeatability_zernikes.jpg
    :target: ../_static/CWFS_repeatability_zernikes.jpg
    :alt: fig-CWFS_repeatability_zernikes.jpg

    In the CWFS collimation and focus script, the success criteria corresponds to when the defocus, and coma measured WFE values are less than ~40nm. This is roughly 2 sigma.


Mirror Motion from datasets from July 7, 2021
=============================================
.. _July7Notebook: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_20210607/Hexapod_LUT_determination-2021-07-07.ipynb?_xsrf=2%7C30ffc635%7C117a70f51a2cb57c083aec021346b1d6%7C1625867799

Reduced using July7Notebook_.

Took data going up and down in elevation in attempts to build a LUT for the hexapod and a new pointing model.
Note that the CWFS corrections were always applied.
This is basically the same exercise as we performed on July 6th except now with more mirror telemetry and a better pointing model.

.. figure:: /_static/hexapod_LUT_generation_2021-07-07-M1positions.jpg
    :name: fig-hexapod_LUT_generation_2021-07-07-M1positions.jpg
    :target: ../_static/hexapod_LUT_generation_2021-07-07-M1positions.jpg
    :alt: fig-hexapod_LUT_generation_2021-07-07-M1positions.jpg

    Measurements taken to derive a LUT for the hexapod taken with the M1 position measurement system in place.
    There is significant scatter in the hexapod positions as well as a systematic offset (in X) between going down in elevation and coming back up.

.. figure:: /_static/hexapod_LUT_generation_2021-07-07-WFE_measurements.jpg
    :name: fig-hexapod_LUT_generation_2021-07-07-WFE_measurements.jpg
    :target: ../_static/hexapod_LUT_generation_2021-07-07-WFE_measurements.jpg
    :alt: fig-hexapod_LUT_generation_2021-07-07-WFE_measurements.jpg

    CWFS measurement showing the residual error when the model converged.

Conclusions based on data to date
----------------------------------

#. A very significant tip/tilt of M1 is occurring that shows evidence of hysteresis.
   This is correlated with the shape of the M2 hexapod motions and therefore the effect is being partially corrected.
   This dataset does not directly capture the pointing error since the collimation was redone for each position.

#. The sharp features in the motions mean polynomial fit to a LUT will never work very well.
   The issue needs to be fixed at the source (M1) and the hexapod LUTs remade.

#. Non-repeatability in M1 positions (mostly tip/tilt but also translation) will also result in "noise" in the hexapod LUT data, and therefore create a "scatter" that will never be improved.

#. The WFE shows significant astigmatism, except we have no way to control this.
   The trefoil is fairly minimal but it's unclear if there is a trend or this is noise dominated.


Stellar Motion due to Mirror Motion from datasets from July 8, 2021
===================================================================
.. _July8Notebook: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_20210607/Up-Down-Repeatability-Test-2021-07-08.ipynb?_xsrf=2%7C30ffc635%7C117a70f51a2cb57c083aec021346b1d6%7C1625867799

Reduced using July8Notebook_.

This dataset is the same as the dataset taken on the 7th, except the CWFS measurements are taken and no correction in pointing or collimation is applied.
The point of this test is to directly measure the stellar position offsets and WFE induced by the non-repeatability in the mirror tilt.
Because the LUT in the hexapod is still active, only residuals are measured.


.. figure:: /_static/Up-Down-Repeatability-Test-2021-07-08-Image_offsets.jpg
    :name: fig-Up-Down-Repeatability-Test-2021-07-08-Image offsets.jpg
    :target: ../_static/Up-Down-Repeatability-Test-2021-07-08-Image_offsets.jpg
    :alt: fig-Up-Down-Repeatability-Test-2021-07-08-Image_offsets.jpg

    Measurements taken to using the LUT for the hexapod with the M1 position measurement system in place.
    The orange and blue curves show two separate runs of the same dataset.

.. figure:: /_static/Up-Down-Repeatability-Test-2021-07-08-WFE.jpg
    :name: fig-Up-Down-Repeatability-Test-2021-07-08-WFE.jpg
    :target: ../_static/Up-Down-Repeatability-Test-2021-07-08-WFE.jpg
    :alt: fig-Up-Down-Repeatability-Test-2021-07-08-WFE.jpg

    CWFS measurement showing the measured error at each elevation.
    No corrections from these measurements were applied.


Conclusions based on data to date
---------------------------------

#. The tilts in M1 and the image motion seen on the sensor are very similar in shape and size.
   Even the hysteresis is seen.
   This is very strong evidence that the M1 motion is causing the pointing non-repeatability.

#. The WFE shows large amount of astigmatism and the trefoil curves have different shapes.
   This makes me inclined to believe there is inaccuracies in the WFE outputs being derived for (at least) these Zernikes.


Closed-Dome Pressure Reduction tests from July 15, 2021
=======================================================
.. _July15Notebook: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_20210607/M1_Mirror_motion-ClosedDomeTests-2021-07-15.ipynb

Reduced using July15Notebook_.


On July 15th, a series of closed-dome tests were performed to look at how reducing the pressure under M1 relates to the mirror motion.
It was determined that a reduction in pressure by 10% of the maximum results in the mirror not tilting (or lifting) a significant amount.

.. figure:: /_static/M1_Mirror_motion-ClosedDomeTests-2021-07-15_10percent.jpg
    :name: fig-M1_Mirror_motion-ClosedDomeTests-2021-07-15_10percent.jpg
    :target: ../_static/M1_Mirror_motion-ClosedDomeTests-2021-07-15_10percent.jpg
    :alt: fig-M1_Mirror_motion-ClosedDomeTests-2021-07-15_10percent.jpg

    An offset by 10% of the maximum pressure resulted in no mirror lifting.
    Note that the hexapod positions (the center row of plots) is not indicative of anything as it is simply following the loaded LUT.

On-Sky Pressure Reduction tests from July 27, 2021
==================================================
.. _July27Notebook: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_20210607/Pressure_reduction_Test-2021-07-27.ipynb

Reduced using July27Notebook_.

A very quick on-sky test was performed to look at image quality for the reduced pressure.
Collimation and Focus were performed at each position since it was known the hexapod LUTs would no longer apply.
The following plot shows how the motions of M1 and hexapod are greatly reduced.
By eye, the images looked reasonable, but no detailed analysis was performed as we know this to be a (very) non-ideal and unrealistic LUT.

.. figure:: /_static/Pressure_reduction_initial_on_sky_test.jpg
    :name: fig-Pressure_reduction_initial_on_sky_test.jpg
    :target: ../_static/Pressure_reduction_initial_on_sky_test.jpg
    :alt: fig-Pressure_reduction_initial_on_sky_test.jpg

    This set of tests demonstrates that a reduction in pressure may lead to a more stabilized M1 and possibly not require a significant decrease in image quality.
    Clearly a more detailed test is required.

Tensioning the Top-end Spider Vanes, August 12, 2021
====================================================

.. _FEA_analysis: https://docushare.lsstcorp.org/docushare/dsweb/Services/Document-37930

Based on an FEA_analysis_ by Myung Cho, the top-end spider vanes were torqued to their proper values in order to ensure no top-end motion should occur.
Ideally this would mitigate both any suspected windshake and hysteresis due to the vanes going between compression and tension.

During the tensioning, performed by Roberto Tighe and Mario Rivera, no tilting of the hexapod was observed.
It is expected that the alignment for the run should be pretty close to what it was previously.

Deriving a new M1 Look-up Table
===============================
.. _M1_mirror_testing-2021-08-09: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_20210607/M1_mirror_testing-2021-08-09.ipynb

.. _M1_Mirror_motion-ClosedDomeTests-2021-08-09: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_202108/M1_Mirror_motion-ClosedDomeTests-2021-08-09.ipynb

The data discussed in this section was taken using M1_mirror_testing-2021-08-09_ and reduced using M1_Mirror_motion-ClosedDomeTests-2021-08-09_.

The original M1 LUT was derived according the proceedure detailed in `TSTN-012 <https://tstn-012.lsst.io/>`_.
In this new M1 LUT, the pressure is determined by looking at what the maximum pressure can be applied before lifting the mirror (and therefore seeing a large tilt).
In short, a series of runs moving the telescope over the range in elevations at constant pressures offsets from teh LUT were performed, altering the pressure in intervals of ~1% of the maximum pressure in the original LUT.
The invervals were determined by using a scale factor which was used to calculate the pressure offset as follows:

.. code-block:: python

    await atcs.rem.ataos.cmd_offset.set_start(m1=starting_pressure*(1-scale_factor))

Therefore a positive scale factor results in a negative offset to the pressure LUT.

Then the piece of each curve right before the mirror lifting extracted as the "best pressure" for each range in elevation.
The following shows the mirror motion for each dip in elevation corresponding to different scale factors.

.. _fig-scalefactors:
.. figure:: /_static/scale_factor_plots.jpg
    :name: fig-scale_factor_plots.jpg
    :target: ../_static/scale_factor_plots.jpg
    :alt: fig-scale_factor_plots.jpg

    This set of tests shows how the mirror lift varies as a function of pressure applied to M1.

By manually selecting which elevation range from each curve, a new M1 LUT can be derived.
The following plot shows what happens when snippets of each curve are used to derive a new pressure versus elevation relationship.
The methodology is to fit a polynomial to the pressure vs elevation relationship, then when this function is used it will result in the mirror motions seen in these plots, except smoothed out since it'll be a continuous function.

.. _fig-piecemeal:
.. figure:: /_static/piecemeal_pressure_plots.jpg
    :name: fig-piecemeal_pressure_plots.jpg
    :target: ../_static/piecemeal_pressure_plots.jpg
    :alt: fig-piecemeal_pressure_plots.jpg

    The amalgamation of selected ranges from the runs shown in Figure fig-scalefactors_.


To derive the new M1 LUT, which is used in the `configuration file <https://github.com/lsst-ts/ts_config_attcs/blob/v0.8.2.alpha.5/ATAOS/v3/m1_hex_20210817_v3.yaml>`_ that was used for the following observing runs, the following polynomial fit was used.
Note that the configuration file includes hexapod LUT values derived from data in the following sections.

.. _fig-pressure_vs_elevation:
.. figure:: /_static/pressure_vs_elevation.jpg
    :name: fig-pressure_vs_elevation.jpg
    :target: ../_static/pressure_vs_elevation.jpg
    :alt: fig-pressure_vs_elevation.jpg

    The fit to the newly derived pressure versus elevation relationship based on when the mirror lifts from the hardpoints.

Closed dome tests using this LUT confirmed the performance on 2021-08-12.

Issues at low elevation
-----------------------
.. _M1_Mirror_motion-ClosedDomeTests-2021-08-17: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_202108/M1_Mirror_motion-ClosedDomeTests-2021-08-17.ipynb

Data in this section was analyzed using M1_Mirror_motion-ClosedDomeTests-2021-08-17_.

It has been observed that for elevations below ~27 degrees or so there is an odd behaviour in the measured M1 mirror pressure.
It shows that there are spikes in pressure that occur and therefore lift the mirror off the hardpoints.
This has been traced to the M1 pressure transducer lower pressure limit.
When operated below the limit, then subsequently in the proper range, a spike is seen.
A new pressure transducer with a larger range is now being procured.
For the following runs we have decided to keep the pressure above ~6.5 PSI, which corresponds to an elevation of ~30 (TBR).

.. _fig-transducer_issues:
.. figure:: /_static/transducer_induced_spike.jpg
    :name: fig-transducer_induced_spike.jpg
    :target: ../_static/transducer_induced_spike.jpg
    :alt: fig-transducer_induced_spike.jpg

    The pressure spikes, which leads to the lifting of the mirror, at low elevation due to exceeding the functional ranges of the pressure Omega pressure transducer.


Testing the new LUTs - Aug 17, 2021
===================================

.. _Up-Down-Repeatability-Test-2021-08-17: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_202108/Up-Down-Repeatability-Test-2021-08-17.ipynb

Data in this section was analyzed using Up-Down-Repeatability-Test-2021-08-17_.


The first night of a three night observing run (two nights of which were weathered out), used the new M1 LUT then consisted of:

#. Deriving new hexapod LUTs by performing collimation and focus at multiple elevations

#. Deriving a new pointing model

#. Testing the image quality by performing dips in elevation and observing the PSF FWHM compared to the DIMM measured seeing.

The data used to derive the hexapod LUTs consisted of performing collimation and focus for a range of elevations.
The average PSF was also determined and compared against the measured DIMM seeing.
It should be noted that the seeing data was quite variable throughout the night, but did contain regions of decent seeing.
None of the testing performed during this night made any effort to align with good seeing patches, the observations came with whatever seeing was currently occurring.

.. _fig-DIMM:
.. figure:: /_static/DIMM_data.jpg
    :name: fig-DIMM_data.jpg
    :target: ../_static/DIMM_data.jpg
    :alt: fig-DIMM_data.jpg

    The seeing as measured by the LSST DIMM.

The following figure shows the difference in PSF width and DIMM seeing as a function of the elevation for the dataset used to derive the hexapod LUT.
Because a collimation and focus sequence is run at each position, one would naively expect to see a small offset between the measured DIMM values and measured PSFs from LATISS, combined with a dependence on the elevation.

.. figure:: /_static/2021-08-17_LUT_determination_dataset.jpg
    :name: 2021-08-17_LUT_determination_dataset.jpg
    :target: ../_static/2021-08-17_LUT_determination_dataset.jpg
    :alt: fig-2021-08-17_LUT_determination_dataset.jpg

    The measured offsets between the measured seeing an DIMM measurements are relatively small, but very noisy.
    Surprisingly there is no elevation dependence, however, during the three sequences the seeing values ranged from 0.8 to 1.4 arcseconds if the large outliers (one in red and one in blue, which correspond to the same outliners in this plot) are omitted.

The WFE measured at each position (after collimation and focus) is shown in the following plot.
Again, curious amounts of astigmatism and trefoil appear

.. figure:: /_static/2021-08-17_LUT_determination_dataset_WFE.jpg
    :name: 2021-08-17_LUT_determination_dataset_WFE.jpg
    :target: ../_static/2021-08-17_LUT_determination_dataset_WFE.jpg
    :alt: fig-2021-08-17_LUT_determination_dataset_WFE.jpg

    The measured Zernikes after collimation+focus correction for each elevation for each dataset.
    The coma and defocus (which are controlled by the hexapod) are well corrected, the astigmatism and trefoil values are interesting but suspect to errors.

Verification of the LUTs from Imagery Data
------------------------------------------

.. _Up-Down-Repeatability-Test-2021-08-17-Verification: https://lsst-nts-k8s.ncsa.illinois.edu/nb/user/pingraha/files/develop/ts_notebooks/pingraham/summit_notebooks/AT_202108/Up-Down-Repeatability-Test-2021-08-17-Verification.ipynb

Data in this section was analyzed using Up-Down-Repeatability-Test-2021-08-17-Verification_.

Two dips in elevation were performed taking imagery data (and not CWFS measurements) at multiple elevations.
All corrections were performed using the ATAOS and LUTs only.
Seeing ranged from ~0.85 to ~1.3 and there were no major outliers.

.. figure:: /_static/2021-08-17_verification_dataset.jpg
    :name: 2021-08-17_verification_dataset.jpg
    :target: ../_static/2021-08-17_verification_dataset.jpg
    :alt: fig-2021-08-17_verification_dataset.jpg

    The measured offsets between the measured seeing an DIMM measurements are relatively small, but an azimuthal dependence is observed.

It is possible that the azimuthal dependence comes from mirror hysteresis, but it could also be a result of the atmosphere.
More data in more stable conditions are required to properly identify and diagnose remaining issues.

Verification of the Pointing Data
---------------------------------

The pointing improved very noticably due to the higher repeatability of the measurements since the M1 tilting issues have been mitigated.
An early pointing model based on a small number of stars was loaded and used and showed remarkable improvement.
A new pointing model has been created, based on a larger but still relatively low number of stars, which has a ~5 arcsec RMS.
Unfortunately, this new model could not be verified due to the remainder of the run being weathered out.

Conclusions based on data to date
---------------------------------

#. The tilting in M1 is now greatly reduced and is much smaller in magnitude (~10 arcsec) and hysteresis (~2 arcsec).

#. Any issues from M2 motion are not easily separable using the on-sky data but if issues remain they appear small.

#. There is still suspect behaviour in the CWFS data, particularly in the astigmatism and possibly the trefoil data

#. A lack of azimuthal dependence in data used to derive the LUT suggests mirror and/or dome seeing is dominating the PSF aberrations. This is also consistent with requiring long (~20s) images to get stable PSFs.

#. The pointing has improved significantly but is still requires verification and a more detailed model.

#. The pressure transducer for the M1 pneumatics system requires replacement prior to moving to lower altitudes.



.. Add content here.
    The optical model says that for a 1 mm shift of the mirror, all in one axis, there is ~65 arcsec of image motion. Note that we measure ~0.14 mm of mirror motion... i think.
    For M2, if we displace it by 1mm, the model says we should get 54.7 arcsec of image motion, We measured ~52.5 and ~50.5 arcsec per mm on sky. I'm not sure why there is a discrepancy... (edited)

    Patrick Ingraham  8:33 PM
    So essentially any error we see from the primary mirror motion is below 10 arcseconds, and that would be complete uncorrected. We're obviously doing some amount of correction so i'm sure it's a small fraction of that.

    Patrick Ingraham  8:47 PM
    to get 30 arcsec of image motion, this requires a tilt of ~0.0022 degrees (8 arcsec) on M2. For a 266mm diameter mirror that's 0.33mm of motion at the edge.
    8:49
    my bet is that the top end is loose and we're seeing hysteresis
.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
