<script lang="ts">
    // Dynamic import of all images
    const images = import.meta.glob(
        ["$lib/assets/p*.jpg", "!$lib/assets/p*-small.jpg"],
        {
            eager: true,
            import: "default",
        },
    ) as Record<string, string>;

    const ph = import.meta.glob(["$lib/assets/p*-small.jpg"], {
        eager: true,
        import: "default",
    }) as Record<string, string>;

    const getImg = (name: string) => {
        const path = `/src/lib/assets/${name}.jpg`;
        return images[path];
    };

    const getPh = (name: string) => {
        const path = `/src/lib/assets/${name}.jpg`;
        return ph[path];
    };

    let loadedImages = $state<Record<string, boolean>>({});
    const iterations = Array.from(Object.entries(images).keys());

    let activeImage = $state(false);
    let current = $state("-1");
    const expand = (e: MouseEvent | TouchEvent) => {
        e.preventDefault();
        let target = e.target as HTMLElement;
        let li: HTMLLIElement | null = target.closest("li") as HTMLLIElement;
        if (li) {
            console.log("active image", li);
            current = li.dataset?.index ?? "-1";
            document.startViewTransition(() => {
                activeImage = !activeImage;
            });
        }
    };
    const closeOverlay = (e: MouseEvent | TouchEvent | KeyboardEvent) =>
        document.startViewTransition(() => {
            activeImage = false;
        });
</script>

<article class:image-active={activeImage}>
    <div
        class="overlay"
        role="button"
        aria-label="dd"
        tabindex="0"
        onclick={closeOverlay}
        onkeydown={closeOverlay}
    ></div>
    <h1>grid images</h1>
    <ul class="image-grid">
        {#each iterations as i}
            {@render gridImage(
                getImg(`p${i + 1}`),
                getPh(`p${i + 1}-small`),
                i.toString(),
            )}
        {/each}
    </ul>
</article>

{#snippet gridImage(img: string, placeholder: string, index: string)}
    {console.log("img", img)}
    <li class="image-wrapper" data-index={index}>
        <a href="#" role="button" onclick={expand}>
            <img
                class="placeholder-img"
                sizes="100svw"
                srcset=""
                loading="eager"
                src={placeholder}
                alt="p2-small"
            />
            <img
                data-index={index}
                class="img"
                class:big-image={activeImage == true && current == index}
                class:hidden-image={activeImage == true && current != index}
                style="view-transition-name: {activeImage && current != index
                    ? 'none'
                    : `image-${index}`}"
                sizes="100svw"
                srcset=""
                loading="eager"
                class:completed={loadedImages[index] === true}
                src={img}
                onload={(e) => {
                    loadedImages[index] = true;
                }}
                alt="dummy image ${index}"
            />
        </a>
    </li>
{/snippet}

<style lang="scss">
    :root {
        --image-width: 280px;
        --image-height: 410px;
    }
    article {
        margin-top: 1rem;
        margin: 0 auto;
        width: 100%;
        height: fit-content;
        max-width: 900px;
    }
    h1 {
        display: inline-block;
        width: 100%;
        text-align: center;
    }
    .image-grid {
        display: grid;
        grid-template-columns: repeat(
            auto-fit,
            minmax(var(--image-width), 250px)
        );
        column-gap: 1rem;
        row-gap: 1rem;
        justify-content: center;
        place-items: center center;
        width: 100%;
        height: max-content;
    }
    .image-wrapper {
        min-width: var(--image-width);
        max-width: 100%;
        min-height: var(--image-width);
        max-height: var(--image-height);
        height: max-content;
        width: max-content;
        display: block;
    }
    a {
        height: fit-content;
        width: fit-content;
        max-height: inherit;
        max-width: inherit;
        max-width: inherit;
        min-width: inherit;
        display: grid;
        place-items: center;
        place-content: center;
        position: relative;
    }
    .placeholder-img,
    .img {
        display: block;
        object-fit: cover;
        object-position: center center;
        border-radius: 20px;
        max-height: inherit;
        min-height: inherit;
        min-width: inherit;
        max-width: inherit;
        height: fit-content; //max-content
        width: fit-content; //max-content
    }

    .placeholder-img {
        opacity: 1;
        filter: blur(0.5rem);
        transition: all ease-in-out;
        animation: pulse 1.5s ease-in-out infinite;
    }
    .img {
        position: absolute;
        opacity: 0;
        filter: blur(1rem);
        transition-property: opacity, filter;
        transition-timing-function: ease-in-out;
    }

    .completed {
        opacity: 1;
        filter: blur(0em);
        transition-property: opacity, filter;
        transition-duration: 800ms;
        transition-timing-function: ease-in-out;
    }
    .image-wrapper:has(.completed) {
        .placeholder-img {
            opacity: 0;
            transition: all 100ms ease-in-out;
            animation-fill-mode: forwards;
            animation-play-state: paused;
        }
    }
    .big-image {
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        transition: none;
        object-position: center center;
        z-index: 15;
        max-height: 70svh;
        min-height: 200px;
        max-width: 70svw;
        min-width: 200px;
        height: 100%;
        width: auto;
    }
    .overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background-color: hsla(0, 0%, 20%, 0.856);
        z-index: 5;
        cursor: pointer;
        opacity: 0;
        pointer-events: none;
        view-transition-name: none;
    }
    .image-active .overlay {
        opacity: 1;
        pointer-events: auto;
    }
    .hidden-image {
        opacity: 0.3;
        view-transition-name: none;
    }
    @keyframes pulse {
        0% {
            opacity: 1;
        }
        50% {
            opacity: 0.9;
        }
        100% {
            opacity: 1;
        }
    }
</style>
