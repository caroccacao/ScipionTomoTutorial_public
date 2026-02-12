# Tutorial_01_Fiducial_Exosome Dataset
**Part II - Quick assessment: Automated Tilt Series Alignment and Tomogram reconstruction using IMOD**

**CryoEM Team** 🔵 **Scipion Team** 
Last update: 04 February 2026, 19:32

**keywords:** ["cryo-ET", "Scipion", "IMOD", "Etomo", "AreTomo", "reconstruction"]

**Reference**: [J.R. Kremer 1996](https://doi.org/10.1006/jsbi.1996.0013)
**Plugin**: [scipion-em-imod](https://github.com/scipion-em/scipion-em-imod)

## What you need for Part II (covered in Part I)
- **import tilt-series movies** finished
- **warp – tilt-series motion and ctf estimation** finished
- **imod – x-ray eraser** finished, hot pixels removed
- **tomoviewer – tilt series curation** finished & 2/5 tilt series selected

Now we connect the pre-processing to tilt-series alignment and tomogram reconstruction. We will start with the blue track.
<table>
<tr>
<td align="center">
<img src="Figures/0-Workflow.png" width="100%"<br>
<sub>Workflow covered in this tutorial</sub>
</td>

## 🔵 The IMOD workflow

Below are the main steps of the IMOD-based workflow as implemented in your Scipion project:
- **imod – Tilt-series preprocessing** to prepare the tilt series for alignment and reconstruction 
- **imod – Coarse prealignment**  to roughly align consecutive tilts using cross-correlation to provide a good starting point.
- **imod – Generate fiducial model**  to locate gold fiducial markers used for alignment.
- **imod – Fiducial alignment** to refine the fiducial model by minimizing residual alignment errors across all tilts.
- **imod – Tomogram reconstruction**  to reconstruct the 3D tomogram from the aligned tilt series.
- **imod – CTF correction and final reconstruction**  to apply CTF-related corrections (if used in your pipeline) and generate the final reconstruction output.

## What this tutorial will teach you
<table>
<tr>
<td align="center">
<img src="Movies/6-TS_raw_x-rayeraser2.gif" height="360" alt="motion and CTF corrected data"><br>
<sub>Motion and CTF corrected tilt series</sub>
</td>

<td align="center">
<img src="Movies/11-FineAlign_TS_interpolated2.gif" height="360" alt="Fine Alignment"><br>
<sub>Aligned, interpolated tilt series</sub>
</td>

<td align="center">
<img src="Movies/Tomogram_Tomo3D_200_SIRT.gif" height="360" alt="Tomogram with SIRT"><br>
<sub>Reconstructed tomogram (SIRT)</sub>
</td>
</tr>
</table>

## 🔵 imod – Tilt-series preprocessing

Before alignment and reconstruction, the tilt series must be prepared for IMOD-based processing.  
This step ensures **data compatibility** with IMOD workflows and prepares the stack for correlation-based alignment and fiducial tracking.
Preprocessing is a technical step: the visual appearance of the tilt series before and after preprocessing should remain **largely unchanged**.

### What happens in this step
- conversion of the tilt series into an IMOD-compatible format  
- basic cleanup and formatting (image order, metadata consistency)  
- preparation of the stack for downstream alignment and reconstruction  

The preprocessing step takes a **cleaned tilt series** (after movie alignment, CTF estimation, and X-ray eraser)  
and produces a **preprocessed tilt-series stack** ready for coarse prealignment, fiducial detection, and fiducial-based alignment.

<table>
<tr>
<td width="50%" align="center">
<img src="Figures/7-imod_tiltseries_preprocess.png" height="360"><br>
<sub>Input cleaned tilt series</sub>
</td>
<td width="50%" align="center">
<img src="Movies/Tilt-seriesPreprocess.gif" height="320"><br>
<sub>Output preprocessed tilt-series stack</sub>
</td>
</tr>
</table>

**Note:** Binning (important)
For fiducial tracking, the coarse-aligned stack does **not** need to be at full resolution.  
To reduce computation time and share resources efficiently, IMOD workflows typically operate on **binned data** at this stage.
In this tutorial, the prealigned stack is generated with **binning = 4**. This binned stack is used for subsequent steps.

### What to inspect after preprocessing
Open the output tilt series in **Tomo Viewer** and check the following:
- no missing or duplicated tilts
- correct tilt order (monotonic change in tilt angle)
- reasonable contrast across the tilt range
- no obvious artifacts introduced during preprocessing
- The output stack is binned with `41 x 1440 x 1024, 5.52 Å/px`

<img src="Figures/7-imod_tiltseries_inputoutput.png" width="70%">

If these conditions are met, the dataset is ready for `coarse prealignment`.

## 🔵 imod – Coarse prealignment

Coarse prealignment computes an initial translational alignment between successive tilt images using cross-correlation.  This step estimates **x/y shifts only** and provides a stable starting point for fiducial tracking. Internally, IMOD runs the program `tiltxcorr`. Internally, IMOD runs the program `tiltxcorr`.

### What happens in this step
- cross-correlation between neighboring tilts  
- cosine stretching of images with higher tilt angles  
- estimation of translational shifts in x and y  
- these shifts are applied to the preprocessed tilt-series stack to generate a **coarse-aligned stack**

<table>
<tr>
<td width="55%" align="center">
<img src="Figures/8-imod_coarse_prealignment.png" height="300"><br>
<sub>Input: preprocessed tilt-series stack</sub>
</td>
<td width="45%" align="center">
<img src="Movies/8-CoarsePreAlignment_TS.gif" height="360"><br>
<sub>Output: coarse-aligned (non-interpolated) stack.</sub>
</td>
</tr>
</table>

**Warning:** This protocol is quite sensitive to the tilt axis orientation. A wrong tilt axis orientation can be the suspicious of casting unusual results. If a wrong tilt axis orientation was imported, the tilt axis can be fixed in the tilt axis form of this protocol.

### What to inspect after coarse alignment
Open the coarse-aligned tilt series in **Tomo Viewer** and check:
- major jumps between consecutive tilts are removed
- features appear roughly aligned across the tilt range
- no obvious mis-ordering of tilts

At this stage, alignment does not need to be perfect.  
The goal is a stable starting point for fiducial model generation.

**Note:** The `+ali` flag means an alignment matrix is stored but not applied to the tilt series. To visualize the aligned series, enable "Interpolated" in the TomoViewer. This applies the transformation matrix. The `interp` flag may appear and only reflects visualization settings — it is not required for further processing.
Scipion follows the IMOD convention, where the tilt axis is aligned with the vertical Y-axis.

## 🔵 imod – Generate fiducial model

The coarse-aligned tilt-series stack is used as input and converted into a **fiducial model** containing tracked landmark positions across all tilts. 
This model defines the positions of landmarks across tilts and serves as the basis for accurate, fiducial-based alignment in the next step.

The IMOD protocol `Generate fiducial model` can be applied to both **fiducial-based** and **fiducial-less** datasets:
- in fiducial-based data, landmarks correspond to gold beads  
- in fiducial-less data, landmarks represent tracked image patches

### What happens in this step 👉 Tracking Step: Where are the fiducials in each tilt image?
- Fiducial markers are detected in the coarse-aligned tilt stack
- Fiducial positions are tracked across the entire tilt range
- A landmark model is constructed for refined alignment

<table>
<tr>
<td width="50%" align="center">
<img src="Figures/9-imod_generatefiducialmodel.png" height="360"><br>
<sub>Input: coarse-aligned tilt-series stack</sub>
</td>
<td width="50%" align="center">
<img src="Movies/GenerateFiducialModel2.gif" height="360"><br>
<sub>Output: fiducial model with tracked landmark positions across the tilt range</sub>
</td>
</tr>
</table>

**Notes:** The fiducial model is stored as a `SetOfLandmarkModels` (one model per tilt series).  Typically, a small number (**10–15**) of well-spread and consistently tracked fiducials is sufficient. Poor fiducial tracking will directly reducethe quality of the final alignment. 

### Quality check: is the fiducial model complete?
Open the fiducial model in **IMOD** and check:
- Fiducials are correctly centered on gold beads
- Fiducials are tracked consistently across many tilts
- No obvious jumps or misassigned fiducials
- Use **Beadfixer** to find gaps

## 3dmod – Beadfixer (manual curation)
Opening the output `SetOfLandmarkModels` of the previous `Generate fiducial model` job in IMOD reveals a small set (~20–25) of well-distributed, pre-tracked fiducials displayed as colored contours. This procedure will display green or purple circles on all the picked gold beads throughout the tilt series. You can switch to `movie` mode and click the middle mouse button to animate through the tilt series and check that beadtrack has done its job, but you must turn 3dmod back into `model` mode before you continue.

As the `SetOfLandmarkModels` is computer-generated, further iterative refinement is required. When imod opens, it opens the tomogram and a 3dmod window, shown below. Select the ZaP Window with the tilt series and press ‘V’ to open a 3D view of the models. For this dataset they should be relatively smooth curves! Turn them by pressing the middle mouse button.

<table>
<tr>
<td width="76%" align="center">
<img src="Figures/10-imod-FiducialAlign_viewmodel.png" height="360"><br>
<sub>Inspecting the fiducial model in 3dmod</sub>
</td>
<td width="24%" align="center">
<img src="Figures/10_SpecialBeadfixer.png" height="360"><br>
<sub>Using the Beadfixer</sub>
</td>
</tr>
</table>

We use the Beadfixer job to check the completeness of the fiducial model. Go to Special: Beadfixer, Select Operation: Fill gaps. Hit `Go to Next Gap`. This will attach you to a point (highlighted with a yellow circle in the ZaP window) that has a missing model point on an adjacent section. Use the Page Up key or the Page Down key (or Z-slider) to go to the previous or next view, which has the missing point. Use the middle mouse button to add the point in the center of the gold particle. It is useful to increase the magnification of the image with the `+` key and adjust the contrast on the sections, especially at high tilts, in order to see the arrows. In case the missing point ends up being outside of the image, the easiest fix is to delete that contour by going back to the section where the point is selected in yellow and click `3dmod:Edit:Contour Delete` or type `Shift+d`. Repeat Go to Next Gap until the message `No more gaps are found` comes up in the main 3dmod window. Save your new model by hitting “S”. Confirm `Done saving model`is written in the 3dmod log window. Close 3dmod, press Done to complete the “Fiducial Model”. 

**Note:** The updated `SetOfLandmarkModels` replaces the previous model when saved (press **S** in 3dmod). Changes such as deleting or modifying contours are permanent and will be reflected in Scipion when the model is reopened. You can verify that your edits were saved by checking the number of contours before and after modification. This step serves as a **manual refinement** of the automatically generated fiducial model.  
The pipeline can run fully automated without user intervention, but alignment quality often improves if fiducial models are inspected and corrected manually. A common workflow is to first run the pipeline automatically, then return to this step to refine the fiducial model and rerun the downstream alignment steps.

> **Legacy:** For a step-by-step manual guide, see the separate protocol by **Emily Machala** [legacy/ETOMO.pdf](legacy/ETOMO.pdf)

## 🔵 imod – Fiducial alignment

Once the landmark (fiducial) models are generated and cleaned, the final alignment of the tilt series can be calculated using the `imod - fiducial alignment` protocol.
In this stage, alignment is refined by correcting:
- Translational shifts (inherited from prealignment)
- Rotational offsets
- This step aligns the tilt axis with the vertical Y-axis following the IMOD convention.

### What this step does 👉 Geometry solving step
This step uses the tracked fiducials to calculate how each tilt image must be shifted and rotated to align the tilt series. 
The protocol `imod – fiducial alignment` uses the `SetOfLandmarkModels` from the previous step to determine the transformation matrix for each tilt image, describing the sample rotation around the Y-axis.

<table>
<tr>
<td width="38%" align="center">
<img src="Figures/10-imod-FiducialAlign1.png" height="360"><br>
<img src="Figures/10-imod-FiducialAlign2.png" height="360"><br>
<sub>IMOD Fiducial alignment input parameters</sub>
</td>
<td width="62%" align="center">
<img src="Figures/11-FineAlign_FixBigResiduals.png" height="360"><br>
<sub>Result: There are not big residuals to fix. Tilt-series alignment successful.</sub>
</td>
</tr>
</table>

### Output of this step
This protocol produces:
- an aligned tilt series `+ali`
- a refined `SetOfLandmarkModels` with interpolated fiducial positions
- a `SetOfCoordinates3D` describing fiducial positions in 3D space
- Use again the beadfixer to help you finding gaps.

<table width="100%">
  <tr>
    <th align="left" colspan="2">
      <b>Tip:</b> The output tilt series as result of the alignment carries an (<code>+ali</code>) flag.
      Open it in the <code>TomoViewer</code> to see the associated transformation matrix per tilt.
      The shown <code>rot</code>, <code>shiftX</code> and <code>shiftY</code> are the rotation and shift extracted from the tilt image following the IMOD convention.
      <br><br>
      The tilt series can be opened with and without the transformation matrix applied (<i>interpolated</i>).
    </th>
  </tr>

  <tr>
    <td align="center" colspan="2">
      <img src="Figures/11-FineAlign_TS_matrix.png" height="100"><br>
      <sub>Transformation matrix per tilt (from <code>TomoViewer</code>)</sub>
    </td>
  </tr>

  <tr>
    <th align="center">Tilt series after fine alignment</th>
    <th align="center">Tilt series after fine alignment (interpolated)</th>
  </tr>

  <tr>
    <td align="center" width="50%">
      <img src="Movies/11-FineAlign_TS.gif" height="360"><br>
    </td>
    <td align="center" width="50%">
      <img src="Movies/11-FineAlign_TS_interpolated2.gif" height="360"><br>
    </td>
  </tr>
</table>


### Optional: Beadfixer to check for big residuals
Part of the process includes reducing the mean residual error to a subpixel value (<1 pixel). The following iterative steps involve fixing fiducial points with large residuals.
Technically `imod – fiducial alignment` reduces that error and interpolates the landmark gaps. Again, it is however helpful to open the Bead Helper from 3dmod main window by Clicking Special:Beadfixer. Is "Go to Next Big Residual" in the Bead Fixer dialog box active? If no, you can close 3dmod. If yes, this will select the model point that had the biggest residual and you may see that it is not centered properly on the gold bead. You can also see a red arrow pointing in the direction of the recommended move (if not, use the zoom controls in the top left corner of the ZaP window or press [+] key until it is big enough for you to see).
•	If you click Move Point by Residual in the Bead Fixer dialog box, it will move the model point by the recommended amount. This works most of the time, but if the suggested position looks wrong, you can move it by manually by centering the cursor in the center of the gold bead and then clicking the right mouse button to shift the point in the right position.
Repeat these two steps until the Go to Next Big Residual shows that no more residuals are found in the 3dmod control window. At this point you can save the model by clicking on Save or type S, and close 3dmod. 
**Note:** There are more options available in the EtomoManual [LINK], not needed in this tutorial. 

## Tomogram reconstruction

There are many methods to reconstruct tomogram in ScipionTomo framework, as they are:
 - Tomo3d
 - Imod
 - NovaCTF
 - AreTomo
 - Emantomo
 - Reliontomo

In this tutorial we will use imod and tomo3D.

## 🔵 First Tomogram reconstruction with imod

At this stage, the optimal tomogram thickness is usually unknown. Therefore, we start with a conservative first reconstruction using 300 voxels (or thicker).
Next, inspect the tomogram in Tomo Viewer or IMOD and evaluate:
- Is the tomogram centered in Z?
- How much empty volume can be cropped from the top and bottom?

For this dataset, we keep the center and crop 50 voxels from both sides, resulting in a final thickness of 200 voxels.

<table width="75%">
  <tr>
    <th align="center" colspan="2">Reconstruction parameters (300 voxels thickness)</th>
  </tr>
  <tr>
    <td align="center" colspan="2">
      <img src="Figures/12-imod_TomogramReconstruction.png" width="60%"><br>
      <sub>Reconstruction parameters (300 voxels thickness)</sub>
    </td>
  </tr>

  <tr>
    <th align="center">Initial reconstruction (300 voxels)</th>
    <th align="center">Cropped reconstruction (200 voxels, centered)</th>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="Movies/Tomogram_300.gif" width="75%"><br>
      <sub>Initial reconstruction (300 voxels)</sub>
    </td>
    <td align="center" width="50%">
      <img src="Movies/Tomogram_imod_200.gif" width="75%"><br>
      <sub>Cropped reconstruction (200 voxels, centered)</sub>
    </td>
  </tr>
</table>

## 🔵 CTF correction

To finalize the tomogram, we apply CTF correction using the protocol `imod – correct CTF`.
This protocol takes as input:
- the **fine aligned tilt series** (with alignment metadata assigned)
- the **CTF estimates** from the WARP motion and CTF estimation step

For this tutorial, keep all parameters at their default values.

<table width="100%">
  <tr>
    <th align="center">CTF correction: input parameters</th>
  </tr>
  <tr>
    <td align="center">
      <img src="Figures/12-imod-CTFcorr.png" width="60%"><br>
      <sub>Input parameters for CTF correction</sub>
    </td>
  </tr>
</table>

## 🔴 Final tomogram reconstruction with tomo3D

**Reference**: [J.I. Agulleiro 2011](https://doi.org/10.1093/bioinformatics/btq692)[J.I. Agulleiro 2015](https://doi.org/10.1016/j.jsb.2014.11.009)
**Plugin**: [scipion-em-tomo3d](https://github.com/scipion-em/scipion-em-tomo3d)

To reconstruct the final tomogram from a **CTF-corrected, aligned tilt series**, we use `tomo3d - reconstruct tomogram`. Tomo3D provides two reconstruction algorithms:

- Weighted Back Projection **(WBP)**
- Simultaneous Iterative Reconstruction Technique **(SIRT)**

### How WBP and SIRT differ (technical intuition)

**WBP (Weighted Back Projection)** is a *single-pass* analytic reconstruction:
- Each 2D projection is **backprojected** into 3D after applying a **frequency-space weighting/filter**.
- **Fast**, and tends to preserve **high-frequency detail**, but may show **streaking artifacts** and lower apparent contrast in noisy cellular data.

**SIRT (Simultaneous Iterative Reconstruction Technique)** is an *iterative* reconstruction:
- Starts from an initial 3D estimate, then repeatedly **forward-projects** and compares to the measured projections.
- The **residual** is **backprojected** to update the volume over multiple iterations.
- Often yields **higher visual contrast** and reduced streaking, but can appear **smoother** (high frequencies may be damped depending on iterations/noise).

> **Rule of thumb:** Use **SIRT** for interpretation/segmentation/template matching; use **WBP** for **STA** and when you want a minimally smoothed reconstruction.

Below we show typical visual differences between **SIRT** and **WBP**.

### Inputs and parameters
The input for reconstruction is the **binned, CTF-corrected tilt series**. In this tutorial we use **SIRT** to obtain a higher-contrast tomogram.  
`Tomogram Thickness` was set to **200 voxels**.

### Output visualization
The reconstructed volume can be inspected via **Analyze results**. Alternatively, right-click the output in the **Summary** panel and select a visualization tool.

<table>
  <tr>
    <th align="center">SIRT reconstruction</th>
    <th align="center">WBP reconstruction</th>
  </tr>
  <tr>
    <td align="center">
      <img src="Figures/13-tomo3Dsirt.png" width="94%">
      <br>
      <em>Input: Tomo3D reconstruction with SIRT (200 voxels)</em>
    </td>
    <td align="center">
      <img src="Figures/13-tomo3DWBP.png" width="100%">
      <br>
      <em>Input: Tomo3D reconstruction with WBP (200 voxels)</em>
    </td>
  </tr>
  <tr>
    <td align="center">
      <img src="Movies/Tomogram_Tomo3D_200_SIRT.gif" width="83%">
      <br>
      <em>Animated Tomogram (SIRT)</em>
    </td>
    <td align="center">
      <img src="Movies/Tomogram_Tomo3D_200_WBP.gif" width="100%">
      <br>
      <em>Animated Tomogram (WBP)</em>
    </td>
  </tr>
</table>

## 🔴 Tomogram denoising with Tomo3D (for a segmentation friendly volume)

For segmentation, a reconstructed tomogram is often still very noisy. Applying denoising can improve the visibility of membranes and cellular features and can make manual or ML-based segmentation more robust.

<table>
  <tr>
    <th align="center" colspan="2">Input</th>
  </tr>
  <tr>
    <td align="center" colspan="2">
      <img src="Figures/14-denoise.png" width="60%">
    </td>
  </tr>

  <tr>
    <th align="center">SIRT</th>
    <th align="center">Denoise</th>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="Movies/Tomogram_Tomo3D_200_SIRT.gif" width="86%">
    </td>
    <td align="center" width="50%">
      <img src="Movies/14-Denoising.gif" width="90%">
    </td>
  </tr>
</table>


### Output: Sanity check the denoised volume
Open raw vs denoised side-by-side (slice + ortho views).
Check you didn’t lose: membranes “crispness” and thin filaments.
Look for artifacts: ringing, over-smoothing, patchy textures, hallucinated continuity.

> **Note:** Denoising alters image statistics. Use it for **visualization and segmentation**, but keep a **minimally processed reference tomogram** for quantitative workflows (e.g., STA).

## 🔵 Tomogram filtering in IMOD (optional)

In addition to denoising, simple filters that can further improve interpretability for **visual inspection** and **segmentation** (e.g., emphasizing membranes or suppressing high-frequency noise). Filtering is optional and should be used conservatively.

**Why/when to filter**
- Improve interpretability for inspection/segmentation (reduce high-frequency noise).
- Emphasize specific spatial scales (feature-size dependent).
- Avoid heavy filtering for quantitative workflows (e.g., STA). Keep a minimally processed reference.

> **Legacy:** For a step-by-step example in IMOD, see the separate protocol by **Clara Feldmann**, which documents the filter settings (**filter type + parameters**) used in our workflow: [legacy/filtering.pdf](legacy/filtering.pdf)

## 🔴 Tomogram Segmentation
Segmentation turns a tomogram into a label volume (mask) that can be used for quality check, quantification, and downstream analysis. In this tutorial we show two example workflows in Scipion:
* Tardis: membrane segmentation (fast, robust starting point)
* Membrain: membrane segmentation with a different model/behavior (useful for comparison / sometimes cleaner membranes)

From either segmentation mask we can generate surface meshes using MembrainSkeletonize (great for figures and 3D visualization).
> Rule of thumb: quantify on `masks`, make figures with `meshes`.

### Workflow A: Tardis Segmentation (+ Membrain Skeletonize)
**Your input**
Input: denoised tomograms
Target here: membranes only (binary segmentation)

**Create the Mask**
Output: SetOfTomoMasks (one 3D mask per tomogram), usually binary: 0 = background, 1 = membrane

<table>
  <tr>
    <th align="center" colspan="2">
      Input_Tardis: denoised tomograms; we segment only membranes
    </th>
  </tr>
  <tr>
    <td align="center" colspan="2">
      <img src="Figures/15-Tardis_input.png" width="60%">
    </td>
  </tr>

  <tr>
    <th align="center">Output 1 – SetOfTomoMasks (Tardis) in <code>imod</code></th>
    <th align="center">Output 2 – SetOfMeshes (MembrainSkeletonize) in <code>ChimeraX</code></th>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="Movies/15-Tardis_Membranes_TomoMask.gif" width="87%">
    </td>
    <td align="center" width="50%">
     see below how to open it in ChimeraX
      <img src="Figures/16-membrainskeletonize.png" width="80%">
    </td>
  </tr>
</table>

**Convert the Mask into a SetOfMeshes**
Meshes are surface models derived from masks; meshes are used for: 3D visualization + figure rendering + qualitative inspection
To create the mesh, we run MembrainSkeletonize.

### Visualization of the Mesh in ChimeraX
1. Open the segmentation as shown in the video.
<img src="Movies/17-VisualizeSegInChimeraX.gif" width="95%">

2. Overlay the segmentation with the tomogram in ChimeraX
(note: overlay does currently not work properly)

3. Assess the quality of your mesh
- Does the mesh hug the membrane location (not offset)?
- Do you see holes / spikes / “shrink-wrapping” artifacts?
- Mesh density: too dense = heavy files; too sparse = blocky surfaces
- Do you have to much noise?

### Workflow B: Membrain + Membrain Skeletonize
It’s good practice to try a second method for comparison / sanity check. Since we are segmenting membranes, we will use Membrain as an alternative model to Tardis.

> In some datasets Membrain yields smoother / more continuous membrane masks than Tardis. Always verify by overlay QC.

**Create the mask**
Input: Input: denoised tomograms
Output: SetOfTomoMasks (same concept as above, membrain also produces a mask per tomogram)

Here the output is binary, we can proceed directly to skeletonize/mesh generation
convert mask → mesh for ChimeraX/figures.

### Quick comparison: Tardis vs Membrain
Tardis: often fast + solid baseline segmentation for membranes; good first pass.
Membrain: sometimes produces smoother / more continuous membrane masks; good alternative when Tardis struggles.
Best practice: run both on 1–2 tomograms, compare overlays, then decide what you’ll use for the whole dataset.

<table width="100%">
  <tr>
    <td align="center" width="50%">
      <img src="Figures/18-Membrain_Denoised.png" width="95%"><br>
      <sub>Membrain (denoised)</sub>
    </td>
    <td align="center" width="50%">
      <img src="Figures/18-Tardis_Denoise.png" width="95%"><br>
      <sub>Tardis (denoised)</sub>
    </td>
  </tr>
</table>



# A few thoughts:
## 🪼 Why denoising is important (even if this example looks similar)
<img src="Figures/14_Denoise_orNotDenoise_Workflow.png" width="95%">
Below we compare Membrain segmentation on (A) a raw tomogram vs. (B) the denoised version.
In this high-contrast example, you may think: Ha! There is barely a difference!
That can happen when membranes are already very clear and SNR is decent.
So why denoise at all? Because in many cellular tomograms denoising can make a big difference by:
Stabilizing segmentation in low-SNR regions (thin membranes, crowded cytoplasm)
Reducing false positives from noise, ice contamination, fringes, or reconstruction artifacts
Improving continuity of membrane masks (fewer breaks → nicer meshes)
Reducing parameter sensitivity (thresholds/model confidence become less “twitchy”)
Rule of thumb:
If you need to fight the segmentation parameters a lot, denoising usually helps (or you need a different model).
We’ll revisit this with true cellular samples in the next tutorial, where the effect is typically much more obvious.


## Can I get rid of the Fiducials
### 🪼 Fidder - detects and erases fiducials

**Reference**: [TeamTomo](https://github.com/teamtomo/fidder)
**Plugin**: [scipion-em-fidder](https://github.com/scipion-em/scipion-em-fidder)

Fiducials markers were used to tilt series, due to their high contrast. However, the strong signal of the gold beads can introduce artifacts in the reconstruction. Specially, the artifacts can complicate the tomogram interpretation and introduce errors in the use of image processing algorithms as: Picking and sutomogram averaging. These effects can be avoided by erasingthe fiducial markers in the images. To do that the protocol `fidder - detect and erase fiducials` can be used. Fidder uses a U-net (deep learning) trained at 8A/px to segment the fiducials. In a second step, the segmented fiducial markers are substituted with white noise matching the local mean and global standard deviation of the image. Fidder only presents a free parameter, the threshold, which aims to determine probability threshold for deteting the gold beads. An strict value of 0.9 should work with this data set. The figures

** When to do it?**
It is possible to get rid of fiducials early and late in the workflow. In our pipeline, we do it after `imod - fiducial alignment` has finished, just before the first reconstruction.
<img src="Figures/Fidder_1_Workflow1.png" width="50%">
<sub>Removing Fiducials prior to reconstruction</sub>

To execute the protocol the next paramaters are used:
- *Input:*
  - **Tilt series**: `imod - Fiducial alignment`
  - **Threshold**: 0.9
<table>
  <tr>
    <th align="center" colspan="2">
      Input (Fidder): aligned tilt series + threshold
    </th>
  </tr>
  <tr>
    <td align="center" colspan="2">
      <img src="Figures/Fidder.png" width="60%"><br>
      <sub>Input parameters for Fidder</sub>
    </td>
  </tr>

  <tr>
    <th align="center">Output 1 – Tilt series after fiducials are erased</th>
    <th align="center">Output 2 – Tomogram reconstructed from the erased tilt series</th>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="Movies/Fidder_onAlignedTS.gif" width="95%"><br>
      <sub>Aligned tilt series (fiducials removed)</sub>
    </td>
    <td align="center" width="50%">
      <img src="Movies/Fidder_Tomo3D.gif" width="95%"><br>
      <sub>Tomogram after reconstruction</sub>
    </td>
  </tr>
</table>


Of course now, the segmentation will also skip fiducials and the final mesh will look better! Try yourself.

### Cropping the final reconstruction & Perfectionate Tomogram Positioning
<img src="Figures/tomogram_positioning_panels.png" width="95%">
<sub>A simple figure illustrating tomogram positioning relative to the tilt axis and beam direction.</sub>

## 🟢 Aretomo - tilt series align and reconstruct in one step
**Reference**: [S. Zheng 2022](https://doi.org/10.1016/j.yjsbx.2022.100068)
**Plugin**: [scipion-em-aretomo](https://github.com/scipion-em/scipion-em-aretomo)

Use the Scipion protocol `aretomo - tilt series align and reconstruct` to perform alignment and reconstruction in a single run, right after tilt-series curation. 

### Parameters used


AreTomo supports two reconstruction methods:
- WBP (Weighted Back Projection)
- SART (Simultaneous Algebraic Reconstruction Technique)

> Tip: WBP is faster, but SART typically gives higher contrast. For cellular environments and template-matching / subtomogram picking, SART is recommended.

The outputs are:
- aligned tilt series
- reconstructed tomogram
- optional: CTF estimation (if enabled)

<table width="100%">
  <tr>
    <td align="center" colspan="2">
      <img src="Figures/20-aretomoForm.png" width="100%"><br>
      <sub>Input parameters</sub>
    </td>
  </tr>
  <tr>
    <td align="center" width="50%">
      <img src="Movies/20-aretomoWithFiducials.gif" width="100%"><br>
      <sub>Reconstruction with fiducials</sub>
    </td>
    <td align="center" width="50%">
      <img src="Movies/20-AretomoNoFiducials.gif" width="100%"><br>
      <sub>Reconstruction without fiducials (10-slice average)</sub>
    </td>
  </tr>
</table>

> **Tip:** You are done here for an overview/quality-check tomogram; if you want to move on to template matching or subtomogram averaging (STA), estimate the CTF and continue with a CTF-aware workflow (CTF correction + final reconstruction / CTF modeling in STA).

More background on AreTomo: aretomo.md (upcoming)




