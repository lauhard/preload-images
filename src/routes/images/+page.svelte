<script lang="ts">
    import type { MouseEventHandler } from "svelte/elements";

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
    let currentIndex = $state(-1);
    const expand = (e: MouseEvent | TouchEvent) => {
        e.preventDefault();
        let target = e.target as HTMLElement;
        let li: HTMLLIElement | null = target.closest("li") as HTMLLIElement;
        if (li) {
            console.log("active image", li);
            currentIndex = parseInt(li.dataset?.index ?? "-1");
            if (document.startViewTransition) {
                document.startViewTransition(() => {
                    activeImage = !activeImage;
                });
            } else {
                activeImage = !activeImage;
            }
        }
    };

    const closeActiveImage = (e: MouseEvent | TouchEvent | KeyboardEvent) => {
        e.preventDefault();
        if (document.startViewTransition) {
            document.startViewTransition(() => {
                activeImage = false;
            });
        } else {
            activeImage = false;
        }
    };

    const nextImage = (e: MouseEvent | TouchEvent) => {
        e.stopPropagation();
        if (currentIndex < iterations.length - 1)
            currentIndex = currentIndex + 1;
        else currentIndex = 0;
    };
    const prevImage = (e: MouseEvent | TouchEvent) => {
        e.stopPropagation();
        if (currentIndex > 0) currentIndex = currentIndex - 1;
        else currentIndex = iterations.length - 1;
    };
</script>

<article class:image-active={activeImage}>
    <div
        class="overlay"
        role="button"
        aria-label="overlay"
        tabindex="0"
        onclick={closeActiveImage}
        onkeydown={closeActiveImage}
    >
        <h2 class="h1">Details</h2>
        <button class="navigation" onclick={nextImage}> next </button>
        <button class="navigation" onclick={prevImage}> prev </button>
    </div>
    <h1>Team Mates</h1>
    <ul class="image-grid">
        {#each iterations as i}
            {@render gridImage(
                getImg(`p${i + 1}`),
                getPh(`p${i + 1}-small`),
                i,
            )}
        {/each}
    </ul>
</article>

{#snippet gridImage(img: string, placeholder: string, index: number)}
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
                class:big-image={activeImage == true && currentIndex == index}
                class:hidden-image={activeImage == true &&
                    currentIndex != index}
                style="view-transition-name: {activeImage &&
                currentIndex != index
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
        --image-height: 280px;
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
    article.image-active {
        h1 {
            opacity: 0;
        }
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
        height: var(--image-height); //max-content;
        width: max-content;
        display: block;
    }
    a {
        height: inherit;
        width: inherit;
        max-height: inherit;
        max-width: inherit;
        max-width: inherit;
        min-width: inherit;
        display: grid;
        place-items: center;
        place-content: center;
        position: relative;
        -webkit-tap-highlight-color: transparent;
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
        height: inherit; //fit-content; //max-content
        width: fit-content; //max-content
    }

    .placeholder-img {
        opacity: 1;
        filter: blur(0.5rem);
        transition: opacity 100ms ease-in-out;
        animation: pulse 1.5s ease-in-out infinite;
        will-change: opacity;
    }
    .img {
        position: absolute;
        opacity: 0;
        filter: blur(1rem);
        transition: none;
        will-change: transform, opacity;
    }

    .completed {
        opacity: 1;
        filter: blur(0em);
        transition:
            opacity 400ms ease-in-out,
            filter 400ms ease-in-out;
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
        max-height: 60svh;
        min-height: 200px;
        max-width: 60svw;
        min-width: 200px;
        height: 100%;
        width: auto;
        -webkit-tap-highlight-color: transparent;
        -webkit-touch-callout: none;
        -webkit-user-select: none;
        user-select: none;
        pointer-events: none;
        will-change: transform;
        @media (width < 900px) {
             max-width: 95svw;
             max-height: 75svh;
             width: 100%;
             height: auto;
        }
    }
    .overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100svw;
        height: 100svh;
        margin: 0;
        padding: 0;
        background-color: hsla(0, 0%, 6%, 0.931);
        z-index: 5;
        cursor: pointer;
        opacity: 0;
        pointer-events: none;
        view-transition-name: none;
        -webkit-tap-highlight-color: transparent;
        -webkit-touch-callout: none;
        -webkit-user-select: none;
        user-select: none;
        transition: opacity 200ms ease-out;
        will-change: opacity;
    }
    .overlay > h2 {
        color: white;
        margin-top: 8svh;
        display: inline-block;
        width: 100%;
        text-align: center;
    }
    .overlay > .navigation {
        position: fixed;
        top: 50%;
        transform: translateY(-50%);
        border-color: white;
        color: white;
        &:nth-of-type(1) {
            left: calc((100svw / 2) + 30svw);
        }
        &:nth-of-type(2) {
            right: calc((100svw / 2) + 30svw);
        }
        @media (width < 900px) {
            top: 95%;
            transform: translateY(-90%);
            &:nth-of-type(1) {
                left: 52%;
            }
            &:nth-of-type(2) {
                right: 52%;
            }
        }
    }
    
    .image-active .overlay {
        opacity: 1;
        pointer-events: auto;
    }
    .hidden-image {
        opacity: 0.3;
        view-transition-name: none;
    }
    .navigation {
        display: block;
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
