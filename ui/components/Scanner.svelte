<script>
    import { Plugins } from '@capacitor/core';
    import { createEventDispatcher, onMount } from 'svelte';
    import QrScanner from 'qr-scanner';

    import { __ANDROID__, __WEB__ } from '~/lib/platform';

    QrScanner.WORKER_PATH = '/scanner.worker.min.js';

    const dispatch = createEventDispatcher();

    let video;
    let scanner;
    let camera;
    let cameraError = false;

    const scannerMobile = async (init) => {
        const _capture = async () => {
            if (camera) {
                const capture = await camera.capture();
                const img = new Image();
                img.src = `data:image/jpeg;base64,${capture.value}`;
                const data = await QrScanner.scanImage(img);
                if (data) {
                    dispatch('message', data);

                    camera.stop();
                    camera = null;
                } else {
                    requestAnimationFrame(scannerMobile);
                }
            }
        };

        if (typeof init === 'boolean') {
            try {
                const { CameraPreview } = Plugins;
                camera = CameraPreview;
                await camera.start({ position: 'rear' });

                // https://github.com/cordova-plugin-camera-preview/cordova-plugin-camera-preview/issues/433#issuecomment-370483348
                setTimeout(async () => {
                    try {
                        await _capture();
                    } catch (err) {
                        requestAnimationFrame(scannerMobile);
                    }
                }, 500);
            } catch (err) {
                cameraError = true;
            }
        } else {
            try {
                await _capture();
            } catch (err) {
                requestAnimationFrame(scannerMobile);
            }
        }
    };

    const scannerWeb = async (stream) => {
        try {
            navigator.getUserMedia =
                navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;
            if (!navigator.mediaDevices.getUserMedia && !navigator.getUserMedia) {
                cameraError = true;
            } else {
                scanner = new QrScanner(video, (data) => {
                    dispatch('message', data);
                    scanner.destroy();
                    scanner = null;
                });
                await scanner.start();
            }
        } catch (err) {
            cameraError = true;
        }
    };

    onMount(() => {
        if (__WEB__) {
            scannerWeb();
        } else {
            scannerMobile(true);
        }

        return () => {
            if (camera) {
                camera.stop();
                camera = null;
            }
            if (scanner) {
                scanner.destroy();
                scanner = null;
            }
        };
    });
</script>

<style>
    main {
        height: 85vh;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
    }

    main.error {
        background: var(--bg);
    }

    scanner {
        flex: 1;
        position: relative;
        overflow: hidden;
        opacity: 1;
    }

    scanner.enabled {
        opacity: 1;
    }

    svg {
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        margin: auto;
    }

    svg path {
        fill: white;
    }

    .video-container {
        position: absolute;
        top: 0px;
        height: 100%;
        width: auto;
    }

    .video-container-web {
        left: 50%;
        transform: translate(-50%, 0);
    }

    .video-container-android {
        left: 100%;
    }

    video {
        display: block;
        height: 100%;
    }

    p {
        text-align: center;
        vertical-align: middle;
        line-height: 85vh;
        font-family: 'Metropolis', sans-serif;
        font-size: 6vw;
        color: #131f37;
    }
</style>

<main class:error="{cameraError}">
    {#if cameraError}
        <p>Camera not authorised</p>
    {:else}
        <scanner class:enabled="{scanner}">
            <div class="video-container" class:video-container-web="{__WEB__}" class:video-container-android="{__ANDROID__}">
                <video bind:this="{video}" autoplay playsinline></video>
            </div>
            <svg width="204" height="204" xmlns="http://www.w3.org/2000/svg">
                <path
                    d="M167 10V0h26.976c5.523 0 10 4.477 10 10v27h-10V10H167zM36.976 10H10v27H0V10C0 4.477 4.477 0 10
                    0h26.976v10zM167 194h26.976v-27h10v27c0 5.523-4.477 10-10 10H167v-10zm-130.024 0v10H10c-5.523
                    0-10-4.477-10-10v-27h10v27h26.976z"
                ></path>
            </svg>
        </scanner>
    {/if}

</main>
