<script>
    import { onMount } from 'svelte'

    const variants = [
        { key: 'current', name: 'Layout Atual' },
        { key: 'netflix', name: 'Padrão Netflix' }
    ]
    const variantClasses = ['prototype-netflix']
    const supportedKeys = variants.map(v => v.key)

    let enabled = $state(false)
    let variantKey = $state('current')

    const isEditableTarget = target => {
        if (!target) return false
        if (target.isContentEditable) return true
        const tag = (target.tagName || '').toUpperCase()
        return tag === 'INPUT' || tag === 'TEXTAREA' || tag === 'SELECT'
    }

    const normalizeVariant = value => {
        const raw = String(value || '').toLowerCase()
        if (raw === 'netflix' || raw === 'netflix-a' || raw === 'a') return 'netflix'
        if (raw === 'current') return 'current'
        return 'current'
    }

    const shouldEnablePrototype = () => true

    const syncVariantClass = key => {
        if (!document?.body) return
        document.body.classList.remove(...variantClasses)
        if (key === 'netflix') {
            document.body.classList.add('prototype-netflix')
        }
    }

    const writeURL = key => {
        const url = new URL(window.location.href)
        url.searchParams.set('prototype', key)
        url.searchParams.set('variant', key)
        history.replaceState({}, '', url.toString())
    }

    const selectVariant = key => {
        variantKey = normalizeVariant(key)
        writeURL(variantKey)
        syncVariantClass(variantKey)
    }

    const rotateVariant = direction => {
        const currentIndex = supportedKeys.indexOf(variantKey)
        const nextIndex = (currentIndex + direction + supportedKeys.length) % supportedKeys.length
        selectVariant(supportedKeys[nextIndex])
    }

    onMount(() => {
        const params = new URLSearchParams(window.location.search)
        enabled = shouldEnablePrototype()
        if (!enabled) return

        variantKey = normalizeVariant(params.get('prototype') || params.get('variant'))
        writeURL(variantKey)
        syncVariantClass(variantKey)

        const onKeyDown = event => {
            if (!enabled || isEditableTarget(document.activeElement)) return
            if (event.key === 'ArrowLeft') {
                event.preventDefault()
                rotateVariant(-1)
            } else if (event.key === 'ArrowRight') {
                event.preventDefault()
                rotateVariant(1)
            }
        }

        window.addEventListener('keydown', onKeyDown)
        return () => {
            window.removeEventListener('keydown', onKeyDown)
            document?.body?.classList?.remove(...variantClasses)
        }
    })
</script>

{#if enabled}
<div class="prototype-switcher" aria-label="Prototype variant selector">
    <button type="button" onclick={() => rotateVariant(-1)} aria-label="Variant previous">←</button>
    <span>{variants.find(v => v.key === variantKey)?.name}</span>
    <button type="button" onclick={() => rotateVariant(1)} aria-label="Variant next">→</button>
</div>
{/if}

<style>
.prototype-switcher {
    position: fixed;
    left: 50%;
    bottom: 2.2vmin;
    transform: translateX(-50%);
    z-index: 99998;
    display: inline-flex;
    gap: 0.5rem;
    align-items: center;
    padding: 0.45rem 0.8rem;
    border-radius: 999px;
    border: 1px solid rgba(255, 255, 255, 0.35);
    background: rgba(7, 7, 9, 0.88);
    color: #fff;
    font-size: 0.85rem;
    line-height: 1;
    box-shadow: 0 0.5rem 1.25rem rgba(0, 0, 0, 0.45);
    backdrop-filter: blur(3px);
}

.prototype-switcher button {
    border: 0;
    width: 2rem;
    height: 2rem;
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.15);
    color: inherit;
    cursor: pointer;
    font-size: 1rem;
}

.prototype-switcher button:hover,
.prototype-switcher button:focus {
    background: rgba(255, 255, 255, 0.28);
}
</style>
