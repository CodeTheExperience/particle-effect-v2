import React, { useEffect, useRef } from "react"
import { addPropertyControls, ControlType } from "framer"

interface ParticleProps {
    x: number
    y: number
    effect: Effect
}

class Particle {
    originX: number
    originY: number
    effect: Effect
    x: number
    y: number
    ctx: CanvasRenderingContext2D
    size: number
    color: string
    vx: number = 0
    vy: number = 0
    force: number = 0
    angle: number = 0
    distance: number = 0
    friction: number = 0.95
    ease: number = 0.1

    constructor({ x, y, effect }: ParticleProps) {
        this.originX = this.x = Math.floor(x)
        this.originY = this.y = Math.floor(y)
        this.effect = effect
        this.ctx = this.effect.ctx
        this.size = 2 // Fixed size of 2 pixels
        this.color = "black" // Fixed color to black
    }

    draw() {
        this.ctx.fillStyle = this.color
        this.ctx.fillRect(this.x, this.y, this.size, this.size)
    }

    update(mouseX: number, mouseY: number) {
        const dx = mouseX - this.x
        const dy = mouseY - this.y
        this.distance = Math.sqrt(dx * dx + dy * dy)
        this.force = -this.effect.mouse.radius / this.distance

        if (this.distance < this.effect.mouse.radius) {
            this.angle = Math.atan2(dy, dx)
            this.vx += this.force * Math.cos(this.angle)
            this.vy += this.force * Math.sin(this.angle)
        }

        this.x +=
            (this.vx *= this.friction) + (this.originX - this.x) * this.ease
        this.y +=
            (this.vy *= this.friction) + (this.originY - this.y) * this.ease

        this.draw()
    }
}

class Effect {
    width: number
    height: number
    ctx: CanvasRenderingContext2D
    particlesArray: Particle[] = []
    gap: number = 20
    mouse: { x: number; y: number; radius: number } = {
        x: 0,
        y: 0,
        radius: 150,
    }

    constructor(width: number, height: number, ctx: CanvasRenderingContext2D) {
        this.width = width
        this.height = height
        this.ctx = ctx
        this.init()
    }

    init() {
        this.particlesArray = []
        for (let y = 0; y < this.height; y += this.gap) {
            for (let x = 0; x < this.width; x += this.gap) {
                this.particlesArray.push(new Particle({ x, y, effect: this }))
            }
        }
    }

    update() {
        this.ctx.clearRect(0, 0, this.width, this.height)
        for (let particle of this.particlesArray) {
            particle.update(this.mouse.x, this.mouse.y)
        }
    }
}

interface ParticleWaveEffectProps {
    particleGap: number
    children: React.ReactNode
}

export function ParticleWaveEffect({
    particleGap,
    children,
}: ParticleWaveEffectProps) {
    const canvasRef = useRef<HTMLCanvasElement>(null)
    const effectRef = useRef<Effect | null>(null)
    const containerRef = useRef<HTMLDivElement>(null)

    useEffect(() => {
        const canvas = canvasRef.current
        if (!canvas) return

        const ctx = canvas.getContext("2d")
        if (!ctx) return

        const updateCanvasSize = () => {
            canvas.width = window.innerWidth * window.devicePixelRatio
            canvas.height = window.innerHeight * window.devicePixelRatio
            canvas.style.width = `${window.innerWidth}px`
            canvas.style.height = `${window.innerHeight}px`
        }

        updateCanvasSize()

        effectRef.current = new Effect(canvas.width, canvas.height, ctx)
        effectRef.current.gap = particleGap

        const handleMouseMove = (e: MouseEvent) => {
            if (effectRef.current) {
                const rect = canvas.getBoundingClientRect()
                effectRef.current.mouse.x =
                    (e.clientX - rect.left) * window.devicePixelRatio
                effectRef.current.mouse.y =
                    (e.clientY - rect.top) * window.devicePixelRatio
            }
        }

        const handleResize = () => {
            updateCanvasSize()
            if (effectRef.current) {
                effectRef.current.width = canvas.width
                effectRef.current.height = canvas.height
                effectRef.current.init()
            }
        }

        window.addEventListener("mousemove", handleMouseMove)
        window.addEventListener("resize", handleResize)

        const animate = () => {
            if (effectRef.current) {
                effectRef.current.update()
            }
            requestAnimationFrame(animate)
        }

        animate()

        return () => {
            window.removeEventListener("mousemove", handleMouseMove)
            window.removeEventListener("resize", handleResize)
        }
    }, [particleGap])

    return (
        <div
            ref={containerRef}
            style={{
                overflow: "hidden",
                height: "100%",
                position: "relative",
                backgroundColor: "white",
            }}
        >
            <canvas
                ref={canvasRef}
                style={{ position: "absolute", top: 0, left: 0 }}
            />
            {children}
        </div>
    )
}

addPropertyControls(ParticleWaveEffect, {
    particleGap: {
        type: ControlType.Number,
        defaultValue: 20,
        min: 10,
        max: 50,
        step: 1,
        title: "Particle Gap",
    },
})
