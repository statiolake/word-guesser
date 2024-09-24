<script lang="ts">
	import { onDestroy, onMount } from 'svelte';

	export let numberOfPieces: number = 150;

	let canvasElement: HTMLCanvasElement;
	let animationFrameId: number;

	interface ConfettiPiece {
		x: number;
		y: number;
		z: number;
		size: number;
		color: string;
		speed: number;
		rotationX: number;
		rotationY: number;
		rotationZ: number;
		rotationSpeedX: number;
		rotationSpeedY: number;
		rotationSpeedZ: number;
		wobble: number;
	}

	let confettiPieces: ConfettiPiece[] = [];
	const colors: string[] = ['#f00', '#0f0', '#00f', '#ff0', '#f0f', '#0ff'];

	function createConfettiPieces(): void {
		confettiPieces = [];
		for (let i = 0; i < numberOfPieces; i++) {
			confettiPieces.push({
				x: Math.random() * window.innerWidth,
				y: Math.random() * window.innerHeight - window.innerHeight,
				z: Math.random() * 200 - 100,
				size: Math.random() * 5 + 5,
				color: colors[Math.floor(Math.random() * colors.length)],
				speed: Math.random() * 1 + 0.5,
				rotationX: Math.random() * Math.PI,
				rotationY: Math.random() * Math.PI,
				rotationZ: Math.random() * Math.PI,
				rotationSpeedX: (Math.random() - 0.5) * 0.1,
				rotationSpeedY: (Math.random() - 0.5) * 0.1,
				rotationSpeedZ: (Math.random() - 0.5) * 0.1,
				wobble: Math.random() * 0.2
			});
		}
	}

	function rotateX(angle: number): number[] {
		const cos = Math.cos(angle);
		const sin = Math.sin(angle);
		return [1, 0, 0, 0, cos, -sin, 0, sin, cos];
	}

	function rotateY(angle: number): number[] {
		const cos = Math.cos(angle);
		const sin = Math.sin(angle);
		return [cos, 0, sin, 0, 1, 0, -sin, 0, cos];
	}

	function rotateZ(angle: number): number[] {
		const cos = Math.cos(angle);
		const sin = Math.sin(angle);
		return [cos, -sin, 0, sin, cos, 0, 0, 0, 1];
	}

	function multiplyMatrices(a: number[], b: number[]): number[] {
		return [
			a[0] * b[0] + a[1] * b[3] + a[2] * b[6],
			a[0] * b[1] + a[1] * b[4] + a[2] * b[7],
			a[0] * b[2] + a[1] * b[5] + a[2] * b[8],
			a[3] * b[0] + a[4] * b[3] + a[5] * b[6],
			a[3] * b[1] + a[4] * b[4] + a[5] * b[7],
			a[3] * b[2] + a[4] * b[5] + a[5] * b[8],
			a[6] * b[0] + a[7] * b[3] + a[8] * b[6],
			a[6] * b[1] + a[7] * b[4] + a[8] * b[7],
			a[6] * b[2] + a[7] * b[5] + a[8] * b[8]
		];
	}

	function applyMatrix(point: number[], matrix: number[]): number[] {
		return [
			point[0] * matrix[0] + point[1] * matrix[1] + point[2] * matrix[2],
			point[0] * matrix[3] + point[1] * matrix[4] + point[2] * matrix[5],
			point[0] * matrix[6] + point[1] * matrix[7] + point[2] * matrix[8]
		];
	}

	function animateConfetti(): void {
		if (!canvasElement) return;
		const ctx = canvasElement.getContext('2d');
		if (!ctx) return;

		ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);

		confettiPieces.forEach((piece) => {
			const rotationMatrix = multiplyMatrices(
				multiplyMatrices(rotateX(piece.rotationX), rotateY(piece.rotationY)),
				rotateZ(piece.rotationZ)
			);

			const points = [
				[-piece.size / 2, -piece.size / 2, 0],
				[piece.size / 2, -piece.size / 2, 0],
				[piece.size / 2, piece.size / 2, 0],
				[-piece.size / 2, piece.size / 2, 0]
			].map((point) => applyMatrix(point, rotationMatrix));

			const projectedPoints = points.map((point) => ({
				x: point[0] + piece.x,
				y: point[1] + piece.y
			}));

			ctx.beginPath();
			ctx.moveTo(projectedPoints[0].x, projectedPoints[0].y);
			for (let i = 1; i < 4; i++) {
				ctx.lineTo(projectedPoints[i].x, projectedPoints[i].y);
			}
			ctx.closePath();

			ctx.fillStyle = piece.color;
			ctx.fill();

			piece.y += piece.speed;
			piece.x += Math.sin(piece.y * 0.1) * piece.wobble;
			piece.rotationX += piece.rotationSpeedX;
			piece.rotationY += piece.rotationSpeedY;
			piece.rotationZ += piece.rotationSpeedZ;

			if (piece.y > canvasElement.height) {
				piece.y = -piece.size;
				piece.x = Math.random() * canvasElement.width;
			}
		});

		animationFrameId = requestAnimationFrame(animateConfetti);
	}

	onMount(() => {
		if (canvasElement) {
			canvasElement.width = window.innerWidth;
			canvasElement.height = window.innerHeight;
			createConfettiPieces();
			animateConfetti();
		}
	});

	onDestroy(() => {
		if (animationFrameId) {
			cancelAnimationFrame(animationFrameId);
		}
	});
</script>

<canvas bind:this={canvasElement} style="position: fixed; top: 0; left: 0; pointer-events: none;"
></canvas>
