# Lecture 3.3 : Protyping at the scanner with MATLAB part 2

Title : Protyping at the scanner with MATLAB part 2

Schedule : June 25, 2020 | 16:00-17:00 

Speaker: Oliver Josephs

Affiliations:
- Wellcome Centre for Imaging Neuroscience, Functional Imaging Laboratory, UCL. https://www.fil.ion.ucl.ac.uk/
- Birkbeck-UCL Centre for Neuroimaging. https://bucni.pals.ucl.ac.uk/

[DRAFT IN PREPARATION]

Real-world integration of Gadgetron and MATLAB with the scanner.

## Context

We are using currently using Gadgetron on five MRI scanners at two imaging neuroscience centres. The typical activity at the centres involves multiple subject studies (e.g. 20 participants) extending over a period of weeks to months per study. Image reconstruction must be performed in real time at the scanner unless huge volumes of raw k-space data are to be accrued, which is only rarely sensible for our neuroscience applications. Several, different, ongoing studies share the same scanners so we need robust separation of applications. For smooth practical operation and for scientific reproducibility a robust, portable and tracable environment is necesary. Docker containers provide a standard environment and isolation between ongoing sequence and reconstruction development and "production" studies. Git repositories on Github serve for tracability.

## Overview

- Hardware and software
- System Integration
- Gadgetron MATLAB software development framework
- Case study: real-time 7T segmented, accelerated 3D EPI
- Parallel processing with Gadgetron MATLAB

## Hardware and software

- Siemens scanners (1.5T Avanto VB17, 3T Prisma VE11C, 7T Terra VE12U)
	- IceGadgetron
	- MaRS (Avanto: MRIR)
- Ethernet networking
	- 1 Gb/s, 10 Gb/s (> 32 channels)
	- Fibre optic for electrical isolation
- Separate Gadgetron PC
  - E.g. Dell T7910 Workstation (now dated)
  - Multiple Cores, large memory
  	- AMD / Intel; Single / Dual socket; >=128 GB
	- https://www.extremetech.com/computing/308501-crippled-no-longer-matlab-2020a-runs-amd-cpus-at-full-speed
	- For robustness one PC per scanner, although could share one PC between scanners for economy
- Ubuntu 20.04 Long Term Support
- Docker 19.03
	- Natively supports NVIDIA gpu card exposure
- Gadgetron 4.1
- gadgetron-matlab 2.0.12
- MATLAB (R2020a)
	- Parallel Computing Toolbox
		- New "thread" pools
- Reconstruction code

## System Integration
- Software traceability
	- _Everything_ in git repositories on GitHub
		- matlab source code, c++ source code, libraries, Dockerfiles, startup scripts, systemctl unit files, xml configurations, ini files, sysctl.conf parameters...
		- Containers built from specific commits. E.g:
```Dockerfile
FROM ubuntu:18.04

# VERSION & REPO TAGS
#-----------------------------------------------------------------------
# ZFP
ARG ZFP_SITE=https://github.com/hansenms/ZFP.git
ARG ZFP_COMMIT=355771b
# INTEL MKL
ARG INTEL_GPG_SITE=https://apt.repos.intel.com/intel-gpg-keys/
ARG INTEL_GPG_FILE=GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB
ARG INTEL_MKL_REPO=https://apt.repos.intel.com/mkl
ARG INTEL_MKL_TAG=2019.4-070
# GADGETRON
ARG GADGETRON_URL=https://github.com/fil-physics/gadgetron
ARG GADGETRON_BRANCH=master
ARG GADGETRON_COMMIT=45035ba
# ISMRMRD
ARG ISMRMRD_URL=https://github.com/ismrmrd/ismrmrd.git
ARG ISMRMRD_COMMIT=a0d2334
ARG ISMRMRD_PYTHON_URL=https://github.com/ismrmrd/ismrmrd-python.git
ARG ISMRMRD_PYTHON_COMMIT=ed11091
ARG ISMRMRD_PYTHON_TOOLS_URL=https://github.com/ismrmrd/ismrmrd-python-tools.git
ARG ISMRMRD_PYTHON_TOOLS_COMMIT=213daf6
ARG SIEMENS_TO_ISMRMRD_URL=https://github.com/ismrmrd/siemens_to_ismrmrd.git
ARG SIEMENS_TO_ISMRMRD_COMMIT=4077e2c
ARG PHILIPS_TO_ISMRMRD_URL=https://github.com/ismrmrd/philips_to_ismrmrd.git
ARG PHILIPS_TO_ISMRMRD_COMMIT=9ef92a1
# BART
ARG BART_REPO=https://github.com/mrirecon/bart.git 
ARG BART_TAG=v0.4.04
.
.
.
```
- Ubuntu security updates
	- Approx. 6 monthly: system disks imaged (serves as backup); all patches applied ```apt full-upgrade```; integration tests; roll back if anything fails.
	- Multiple, near-identical PC's to test for consistency and to swap in if urgently needed.
- Integration tests
	- Gadgetron has nice integration test framework and continuous integration (CI) integrated within GitHub
		- buildbot
	- Raw data and the corresponding expected reconstructed images
	- All test cases should run and the images match (within some limit of precision)
	- Reconstructions sumbitted after publication (peer review) as new integration tests
	- Ensures ongoing compatability
- MATLAB installed on the host but accessbile within the Docker containers
	- For the external language interface 'execute' to work gadgetron has to be able to call a matlab binary (in batch mode).
	- (N.B. line continuation backslashes required at end of lines, common for long Docker commands)

```bash
sudo docker create --name=example_container_name \
  --net=host \
  --privileged \
  -v /hostshare:/hostshare \
  -v /usr/local/MATLAB:/usr/local/MATLAB \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --volume="$HOME/.Xauthority:/root/.Xauthority:rw" \
  -e DISPLAY \
  --gpus all \
  example_image_name
```
and in the Dockerfile, since is not recognised in "docker create" command line(this may be a bug which has been fixed).
```Dockerfile
env NVIDIA_DRIVER_CAPABILITIES=all
```
	
This, last setting gives access to display capabilities of GPU from within the Docker container for Matlab to plot figures. (https://github.com/NVIDIA/nvidia-container-runtime)
  
- Allows for Gadgetron to "execute" MATLAB from withing the Docker container; or to "connect" to MATLAB (inside or) outside docker for debugging.
	- N.B. Matlab may alternatively be installed completely _inside_ the container https://github.com/mathworks-ref-arch/matlab-dockerfile
 		- Mathworks' curated Dockerfiles, etc.
 		- Dependencies, Licensing, Toolboxes, etc.
		- Also intended for cloud deployment.
	- Several, specific MATLAB versions: E.g. R2017b (update 9), R2020a (update 3), R2020b (Prerelease).
	
- Separate Docker containers for different projects. Dockerfiles git version controlled. Based on gadgetron/Docker/base and gadgetron/Docker/incremental.
	- Usually built from source locally, for traceability, with only Ubuntu pulled from DockerHub (e.g. see above example).
	
## Matlab software development framework

- Built on Kristoffer's example reconstruction by extending on 
  - Source tree with recon .m and .xml; +steps and +utils directories.

## Case study: real-time 7T segmented, accelerated 3D EPI

An important feature of Gadgetron is that reconstruction can occur at the scanner. If you have to wait 10 minutes for the reconstruction then there is not such a big benefit using Gadgetron over exporting the data and reconstructing offline. MATLAB has a reputation of being too slow for real-time image reconstruction. But with current multi-core PC hardware we can work around bottlenecks and actually have MATLAB running rather fast. The style of this section will be a top-down  walk-through of code exerpts of an example reconstruction (segmented, accelerated, 3D EPI) to illustrate how we have used and built on the structure that Kristoffer has given us in the gadgetron-matlab examples. I have deliberately only editted the snippets minimally to illustrate "warts and all" the ease with which prototying can be performed.

- Gadgetron chain configuration xml file
	- AcquisitionAccumulateTriggerGadget "n_acquistions" trigger
		- Fixed length buckets for transfer by the external language interface to MATLAB
		- One "Reference" bucket; followed by
		- Multiple "Data" buckets
	- Workaround for the case that trigger counters are not set as expected in sequence
	- Rule of thumb: data blocked and sent <= 10 buckets / second to MATLAB
```xml
.
.
.
        <!-- Human seg 0 caipi 1 pf 6/8 -->
        <gadget>
            <dll>gadgetron_mricore</dll>
            <classname>AcquisitionAccumulateTriggerGadget</classname>
            <property name="trigger_dimension" value="n_acquisitions"/>
            <property>
                <name>n_acquisitions_before_trigger</name>
                <value>10912</value>
            </property>        
            <property>
                <name>n_acquisitions_before_ongoing_trigger</name>
                <value>2068</value>
            </property>        
        </gadget>
        
    <!--
        <external>
            <execute name="epi" type="matlab"/>
            <configuration/>
        </external>
    --> 
	
        <external>
            <connect port="18000"/>
            <configuration/>
        </external>
	
        <gadget>
            <dll>gadgetron_mricore</dll>
            <classname>FloatToUShortGadget</classname>
        </gadget>

```
- overall reconstruction function
	- gadgetron.external.listen called by "execute" or run to "connect"
	- Global structure, g. Passed where necessary to steps
	- Configuration "+utils" functions
	- Switches as part of g or only for debugging configuration
	- Setting up MATLAB Parallel Processing Toolbox parallel pool
	- "+steps" functions
	- Conditional steps chain construction
	- gadgetron.consume
	
```matlab
>> gadgetron.external.listen(18000, @epi);
```

```matlab
function epi(connection)

disp("Matlab EPI reconstruction running.")

%% Generally useful parameters and debugging switches
g = utils.extract_xml_parameters(connection.header);

g.M = utils.generate_TBR_matrix(g);

% Use parallel, data-queued, pipeline if available
g.Parallel = true;

% Use separate processing steps for the acc. acq.'s
Separate_steps = false;

% Produce graphical output
Display = true;

% Save nifti data volumes
Save_nifti = false;

% Send reconstucted images back to client
Send_images = true;

disp(['Num segs = ' num2str(g.SegPE)])
disp(['CAIPI = ' num2str(g.upl('CAIPI'))])


%% Set up parallel thread pool
p=gcp('nocreate');
if isempty(p)
    parpool('threads');
end

%% Set up processing chain

% Get the next data bucket from gadgetron
next = @connection.next;

% Bucket may be a sensitivity reference or an acc. acq. volume

% If it's a sensitivity reference, reconstruct it
next = steps.reconstruct_reference(next, g);

% if it's a sensitity reference volume compute unfolding matrices
next = steps.sense_prepare(next, g);

% Otherwise the bucket is an acc. acq.,

next = steps.extract_data(next, g);

if Separate_steps
    % TODO these do not handle the header correctly, yet
    next = steps.frequency_offset_correction(next, g);
    
    next = steps.reconstruct_1d(next, g);
    
    next = steps.fft2d_and_unfold(next, g);
else
    % To reduce data communication overhead, combine previous three steps
    next = steps.combined_reconstruction(next, g);
end

if Display
    next = steps.display_volume(next);
end

if Save_nifti
    next = steps.save_nifti(next, g);
end

if Send_images
    % Scanner seems to want 2D EPI by slice
    % but 3D EPI by volume
    if g.dimEncod(3) == 1 % 2D
        next = steps.mrm_slices(next, g);
    else % 3D
        next = steps.mrm_volume(next, g);
    end
        
    next = steps.send_to_client(next, connection);
end

%% Execute the processing chain
tic, gadgetron.consume(next); toc

%% End of reconstruction code
end

```
- Example '+steps' function
	- Functions taking and returning functions (function pointers, anyway).
	- Analogous to gadgets in c++ Gadgetron stream.
	- I've used for translating formatting and (parallel process) queueing data for util functions to actually process.
	- globals, closure
	- nested funcion to receive bucket
	- pass-through if not reference
	- +utils and built-ins to reconstruct the reference image
	- "Data" structure, d, for passing between chain of functions or steps
	- passing reference bucket to MATLAB	
```matlab
function next = reconstruct_reference(input, g)
%RECONSTRUCT_REFERENCE Reconstruct fully-encoded sensitivity reference data
%   Detailed explanation goes here

nCha    = g.nCha;                         % No. of coil channels used for acquisition
nFE     = g.dimEncod(1);                  % No. of frequency-encoded points in final image
nFEAcq  = g.upl('numSamples');            % Typically will be nFE * over-sampling factor of 2
nNav    = g.upl('numberOfNavigators');    % No. of phase reference echoes
AccPE   = g.AccFact(2);                   % Acceleration factor of EPI readout
% Why doesn't dimEncod reflect partial Fourier?
% nPEAcq  = g.dimEncod(2) / AccPE;          % No. of lines *acquired* (assuming no partial Fourier) in main (not ref.) EPI readout
Acc3D   = g.AccFact(3);                   % Acceleration factor in partition direction
nParAcq = g.dimEncod(3) / Acc3D;          % No. of partitions acquired

% Processing choices:
nav_smooth          = 10;           % Smoothing to be applied to navigators [k-space span]
nav_padFactor       = 2;            % Padding multiple for extrapolating navigator correction

    function retval = reconstruct_reference(bucket)
        
        if bucket.ref.count == 0 % this is a (hopefully, data) bucket. Don't handle it here.
            retval = bucket;
            return
        end
        
        % N.B. Re. segmentation: For acc. acq. there are g.SegPE segments; for
        % the reference there are g.SegPE * AccPE segments.
        % We assume that the reference segments are acquired sequentially.
        disp(['product=' num2str(nFEAcq*nCha*g.SegPE*AccPE*nParAcq*Acc3D)])
        disp(['size_bucket=' num2str(size(bucket.ref.data))])
        d.data=reshape(bucket.ref.data, nFEAcq, nCha, [], g.SegPE*AccPE, nParAcq, Acc3D);
        %         nRefETL = size(d,3); % Number of lines acquired in EPI Echo train
        
%         %% Frequency Offset Correction
%         % function data = frequency_offset_correction(data, FirstNav_Middle, TE, tAcq, echospaceSec, limits)
        d = utils.frequency_offset_correction(d, g.FirstNav_Middle  * 1e-6,...
            g.TE               * 1e-3,...
            g.tAcq             * 1e-6,...
            g.tEchospace       * 1e-6,...
            g.xml.encoding.encodingLimits.kspace_encoding_step_1);
        
        %% Recon reference
        
        d = utils.recon1d(d, g.M, nNav, nav_padFactor, nav_smooth);
        
        % Partial Fourier
        f1d=zeros([g.dimEncod(2) nFE nCha Acc3D*nParAcq],'like',single(1i));
        f1d(1:size(d.f1d, 1),:,:,:)=d.f1d;
        
        % f1d - Fully sampled reference volume in projection-space after TBR and
        % navigator-based phase-correction; used to calculate sensitivities.
        
        % vvv - Fully sampled reference volume now in image domain having performed
        % FFT along 1st (PE) and 4th (PAR) dimensions:
        vvv=fftshift(fft(fft(fftshift(f1d)),[],4));    % AccPE*nPEAcq, nFE, nCha, Acc3D*nParAcq
        
        % To match undersampled data vol dimension order, for nifti saving, display, etc.
        % nFE, AccPE*nPEAcq, nCha, Acc3D*nParAcq
        vvv=permute(vvv,[2 1 3 4]);
        
        retval=vvv;
        %             save -v7.3 -nocompression ref vvv
        
        % sss - sqrt of sum of squares recon of reference volume:
        %         sss=sqrt(sum(conj(vvv).*vvv,3));
        %         figure(2);montage(sss*7e1);figure(1); % Need to remove for online case
    end
next = @() reconstruct_reference(input());

end
```
- Example '+utils' function
	- 'Normal' (first-order) functions
	- Small utilities or components of recon.
	- Your favourite (1000 line) reconstruction already coded.
	- Analogous to toolboxes in c++ Gadgetron.
	- Might be called directly and / or from steps.
	- "Data" structure, d.
	
	
```matlab
function d = recon1d(d, a1, a2, a3, a4)
%RECON1d TBR 1d reconstuction of a bucket of acquistions

% Only d.data is read, d.f1d created
d.f1d = recon1d(d.data, a1, a2, a3, a4);

    function f1d = recon1d(d, M, nNav, nav_padFactor, nav_smooth)
        
        
        %%
        % Reconstruct aliased image
        %
        [nFEAcq, nCha, nLin, nSeg, nParAcq, Acc3D] = size(d);
        nFE = size(M,1);
        
        % Collect phase and 3D segments dimensions at end
        meas=permute(d,[1 2 5 3 4 6]); %  nFEAcq, nCha, nParAcq, nlin, nSeg, Acc3D
        
        % Merge channels and acquired partions
        meas=reshape(meas,[nFEAcq, nCha*nParAcq, nLin, nSeg, Acc3D]);
        
        if exist('pagemtimes','builtin')
            % R2020b - new built-in function 
            % vectorised matrix multiplication
            f1d = pagemtimes(M(:,:,1:nLin), meas);            
        else
            % f1d: matrix to store the TBR-transformed phase-encoded data in projection domain
            f1d=zeros(nFE, nCha*nParAcq, nLin, nSeg, Acc3D, 'like', single(1i));
            
            % M: [nFE, nFEAcq, nPEAcq];
            % Note: because of M f1d is centred in the FoV (relevant for
            % understanding fftshifts etc)
            for parseg=1:Acc3D
                % Looping over through-plane segments
                for seg=1:nSeg
                    % Looping over in-plane segmented readouts
                    for line = 1 : nLin
                        % Looping over EPI readouts (lines)
                        f1d(:,:,line,seg,parseg)=M(:,:,line)*meas(:,:,line,seg,parseg); % nFE  nCha*nParAcq = [nFE nFEAcq]  * [nFEAcq  nCha*nParAcq], e.g. 128x(52*30) = 128x256 * 256x(52*30)
                    end
                end
            end
        end
 .
 .
 .
    end
 end

```
- [Slide here with graphical summary of closures / control of flow]

- Now concentrate on data flow
- working with data and headers within MATLAB
	- Works but further developmentin progress (6/2020).
	- Typically take geometry from _appropriate_ acquisition to create header for image.
- returning images to the scanner database

## Parallel processing within Gadgetron Matlab

It's difficult to get multiple cores working contunuously with block processing. Often memory allocation / setting (single threaded / slow) interspersed by cpu operation. During single threaded stages other cores available. Solution. Pipeline. Implementation in Kristoffer's functional framework. Parfeval (which NB also does the work for parfor). Simple fifo operation to keep cpus busy and minimise stalling.

- Matlab multi-core processing
 - Naive loop -> optimise loops -> vectorise -> multi-threaded built-ins
 	- Yair Altman "Accelerating MATLAB Performance" https://doi.org/10.1201/b17924
	- Getting a little out of date
  - Implicit parallisation
      - LAPACK
      - Matlab vectorisation
      - MATLAB Parallel Toolbox
      - Kristoffer's functional programming paradigm steps
- IceGadgetron xml configs to get raw data and allow image database receipt
- github ismrmrd mrd handling

### If time permits
- Physiological waveform handling with Gadgetron and MATLAB
- Siemens pulse / ecg / breathing belt
- Timestamping circuitry

## Conclusion
- Have built this with long term on-going collaboration with others.
- If you have Matlab knowledge and develop, please collaborate and feedback.
- Hangouts every Friday, 3pm CET. Videoconference link posted on https://groups.google.com/forum/#!forum/gadgetron.
