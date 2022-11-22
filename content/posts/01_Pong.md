---
title: "Pong with Ebitengine"
date: 2022-10-04T11:20:42-05:00
draft: false

tags: ["go", "ebitengine", "gamedev"]
categories: ["Ebitengine"]
---

## Project Initialization

In this post, we are going to look at my process for recreating pong with Ebitengine in Golang.

As with any Go project, we want to start with initializing a go module and setting up our initial codebase.

```bash
mkdir pong
cd pong 
go mod init github.com/KalebHawkins/pong
touch main.go
```

In our `main.go` file, we will start with a very basic Game interface definition. 

```go
package main

import (
	"log"

	"github.com/hajimehoshi/ebiten/v2"
)

const (
	scrWidth  = 640
	scrHeight = 480
)

type Game struct {
	bg          *ebiten.Image
}

func (g *Game) Update() error {
	return nil
}

func (g *Game) Draw(scr *ebiten.Image) {
	g.bg.Fill(color.White)
	scr.DrawImage(g.bg, nil)

}

func (g *Game) Layout(outWidth, outHeight int) (int, int) {
	return scrWidth, scrHeight
}

func main() {
	g := &Game{
		bg: ebiten.NewImage(scrWidth, scrHeight)
	}

	ebiten.SetWindowTitle("Pong")
	if err := ebiten.RunGame(g); err != nil {
		log.Fatal(err)
	}
}
```

Now we have a 640x480 window with the title of *Pong*. We also have a white background that we draw to the screen. We will be drawing everything else to the background. The next step is to think about what we want our game to do. We should open up a menu and prompt the user with an option to play or quit. If play is hit then we start the game logic and start playing.

## Game States

We are going to have different states where at one point we will be playing the game and at other points, we may be on the menu. Naturally, we will need to track what state we are in and what to do in those states.

```go
// ...

type GameState int

const (
	Menu GameState = iota
	Playing
	Quit
)

type Game struct {
	state       GameState
	bg          *ebiten.Image
}

// ...

func (g *Game) Update() error {
	switch g.state {
	case Menu:
	case Playing:
	case Quit:
		os.Exit(0)
	}

}

func (g *Game) Draw(scr *ebiten.Image) {
	switch g.state {
	case Menu:
	case Playing:
	case Quit:
		os.Exit(0)
	}
}

func main() {
	g := &Game{
		state: Playing
		bg: ebiten.NewImage(scrWidth, scrHeight)
	}

// ...
}
```

Now when our game starts we will automatically start in a playing state. This is so we can get our game mechanics to build out and play with things without going through the menu with every test. 

## The Pong Paddles

We will need two paddles, one for the player and one for the AI. We can represent the paddle as a structure like so.

```go
type Paddle struct {
	img    *ebiten.Image
	pos    *image.Point
	speedY int
}
```

The paddle contains an image for drawing it to the screen, a position, and a speed to move up or down along the Y axis.

The following code adds the player, `paddle`, and ai to the game's structure.

```go
// ...

type Game struct {
	state       GameState
	bg          *ebiten.Image
	paddle 		*Paddle
	ai 			*Paddle
}

// ...

func main() {
	g := &Game{
		state: Playing
		bg: ebiten.NewImage(scrWidth, scrHeight)
		paddle: &Paddle{
			img:    ebiten.NewImage(10, 80),
			pos:    &image.Point{X: 20, Y: (scrHeight / 2)},
			speedY: 7,
		},
		ai: &Paddle{
			img:    ebiten.NewImage(10, 80),
			pos:    &image.Point{X: scrWidth - 20, Y: (scrHeight / 2)},
			speedY: 7,
		},
	}

// ...
}
```

Let's draw them to the screen to see how that looks. I am going to create a `DrawPlaying()` method for this and insert it into the game's draw method.

```go
func (g *Game) DrawPlaying(scr *ebiten.Image) {
	g.bg.Fill(color.White)

	g.paddle.img.Fill(color.Black)
	paddleOpt := &ebiten.DrawImageOptions{}
	paddleOpt.GeoM.Translate(-float64(g.paddle.img.Bounds().Dx()/2), -float64(g.paddle.img.Bounds().Dy())/2)
	paddleOpt.GeoM.Translate(float64(g.paddle.pos.X), float64(g.paddle.pos.Y))

	g.ai.img.Fill(color.Black)
	aiOpt := &ebiten.DrawImageOptions{}
	aiOpt.GeoM.Translate(-float64(g.ai.img.Bounds().Dx()/2), -float64(g.ai.img.Bounds().Dy()/2))
	aiOpt.GeoM.Translate(float64(g.ai.pos.X), float64(g.ai.pos.Y))

	g.bg.DrawImage(g.paddle.img, paddleOpt)
	g.bg.DrawImage(g.ai.img, aiOpt)

	// Draw some lines on the screen to make sure everything lines up right.
	ebitenutil.DrawLine(g.bg, 0, scrHeight/2, scrWidth, scrHeight/2, color.Black)
	ebitenutil.DrawLine(g.bg, scrWidth/2, 0, scrWidth/2, scrHeight, color.Black)

	scr.DrawImage(g.bg, nil)
}

// ...

func (g *Game) Draw(scr *ebiten.Image) {
	switch g.state {
	case Menu:
	case Playing:
		g.DrawPlaying(scr) // <-- Add to Game's Draw method.
	case Quit:
		os.Exit(0)
	}
}
```

Taking a moment to discuss the line below. We do this to translate our image's origin to the center of the image. If we didn't do this our paddle's position x and y values would be where the upper left corner was. That would result in some weird-looking alignment.

```go
paddleOpt.GeoM.Translate(-float64(g.paddle.img.Bounds().Dx()/2), -float64(g.paddle.img.Bounds().Dy())/2)
```

## Player Movement

Alright, time for our player movement. I am going to create a couple of functions. One function handles the "high-level" game logic the other implements the details of that logic. This will work similarly to the `DrawPlaying()` method.

```go
func (g *Game) UpdatePlaying() {
	g.Move(g.paddle)
}

func (g *Game) Move(p *Paddle) {
	if ebiten.IsKeyPressed(ebiten.KeyW) || ebiten.IsKeyPressed(ebiten.KeyA) {
		p.pos.Y -= p.speedY
	}
	if ebiten.IsKeyPressed(ebiten.KeyS) || ebiten.IsKeyPressed(ebiten.KeyD) {
		p.pos.Y += p.speedY
	}
}

func (g *Game) Update() error {
	switch g.state {
	case Menu:
	case Playing:
		g.UpdatePlaying() // <-- Add to Game's Update method.
	case Quit:
		os.Exit(0)
	}

	return nil
}
```

The `W` and `A` keys will move your paddle up. The `S` and `D` keys will move the paddle down. 

## Bounds Checking 

You might have noticed we can move off the screen and don't want that let's add some bounds checking. 

```go
func (g *Game) UpdatePlaying() {
	g.Move(g.paddle)
	g.checkPaddleBounds(g.paddle)
}

// ...

func (g *Game) checkPaddleBounds(p *Paddle) {
	if p.pos.Y-p.img.Bounds().Dy()/2 <= 0 {
		p.pos.Y = p.img.Bounds().Dy() / 2
	}
	if p.pos.Y+p.img.Bounds().Dy()/2 >= scrHeight {
		p.pos.Y = scrHeight - p.img.Bounds().Dy()/2
	}
}
```

That looks pretty cryptic, right? To break it down we take the paddle's Y position (the center of the paddle) and subtract it from half the paddle's height. That will give us a point on the top of the paddle. We check to make sure that point isn't less than or equal to 0. If it is we set the location back to a point where the paddle is on the screen. We do the inverse for the bottom of the screen. These statements will cause the paddle to stop on either the top or bottom of the screen before going off-screen.

## The Ball

Moving up and down gets boring fast. We need a ball to hit. In our case, the ball is going to be a rectangle. If we think about it a ball is similar to the paddles with an extra direction to move. Meaning a ball will have an image, a position, and a speed of X and Y. 

```go
type Ball struct {
	img      *ebiten.Image
	pos      *image.Point
	velocity *image.Point
}
```

> I am treating `image.Point` as a 2d mathematical vector to perform basic movement operations. In larger games you'd likely have a more efficient vector structure or something. 

To get the ball in the game we need to add it to the game struct, draw it, and initialize it. 

```go
type Game struct {
	state       GameState
	bg          *ebiten.Image
	paddle      *Paddle
	ai          *Paddle
	ball        *Ball
}

func (g *Game) DrawPlaying(scr *ebiten.Image) {
	g.bg.Fill(color.White)

	// ...

	g.ball.img.Fill(color.Black)
	ballOpt := &ebiten.DrawImageOptions{}
	ballOpt.GeoM.Translate(-float64(g.ball.img.Bounds().Dx()/2), -float64(g.ball.img.Bounds().Dy()/2))
	ballOpt.GeoM.Translate(float64(g.ball.pos.X), float64(g.ball.pos.Y))

	g.bg.DrawImage(g.paddle.img, paddleOpt)
	g.bg.DrawImage(g.ai.img, aiOpt)
	g.bg.DrawImage(g.ball.img, ballOpt)

	g.bg.DrawImage(g.ball.img, ballOpt)
	// Draw some lines on the screen to make sure everything lines up right.
	ebitenutil.DrawLine(g.bg, 0, scrHeight/2, scrWidth, scrHeight/2, color.Black)
	ebitenutil.DrawLine(g.bg, scrWidth/2, 0, scrWidth/2, scrHeight, color.Black)

	scr.DrawImage(g.bg, nil)
}

func main() {
	g := &Game{
		// ...
		ball: &Ball{
			img:      ebiten.NewImage(10, 10),
			pos:      &image.Point{X: scrWidth / 2, Y: scrHeight / 2},
			velocity: &image.Point{X: rand.Intn(7-3+1) + 3, Y: rand.Intn(6-3+1) + 3},
		},
	}

	ebiten.SetWindowTitle("Pong")
	if err := ebiten.RunGame(g); err != nil {
		log.Fatal(err)
	}
}
```

## Ball Movement

We perform some vector math to simulate ball movement. We also clamp the speed to the max speed. 

```go
func (g *Game) UpdatePlaying() {
	g.Move(g.paddle)
	g.checkPaddleBounds(g.paddle)
	g.ballMove()
}

// ...
func (g *Game) ballMove() {
	g.ball.pos.X += g.ball.velocity.X
	g.ball.pos.Y += g.ball.velocity.Y

	maxSpeed := 20
	if g.ball.velocity.X >= maxSpeed {
		g.ball.velocity.X = maxSpeed
	}
	if g.ball.velocity.X <= -maxSpeed {
		g.ball.velocity.X = -maxSpeed
	}
	if g.ball.velocity.Y >= maxSpeed {
		g.ball.velocity.Y = maxSpeed
	}
	if g.ball.velocity.Y <= -maxSpeed {
		g.ball.velocity.Y = -maxSpeed
	}
}
```

Max speed should probably be part of the ball's attributes however since we are just going to have one ball this is fine. 

![](https://media3.giphy.com/media/QMHoU66sBXqqLqYvGO/giphy.gif)

## Ball Collision

Just like with the paddle we want to check collisions with edge and paddles and such. We are going to start easy and just get edge cases. 

```go
func (g *Game) UpdatePlaying() {
	g.Move(g.paddle)
	g.checkPaddleBounds(g.paddle)
	g.ballMove()
	g.checkBallCollision()
}

// ...
func (g *Game) checkBallCollision() {
	// Top/Bottom collision.
	// Reverse the Y direction.
	if g.ball.pos.Y < 0 || g.ball.pos.Y > scrHeight {
		g.ball.velocity.Y *= -1
	}
	// Left/Right Edge collision.
	// Reset the ball.
	if g.ball.pos.X < 0 || g.ball.pos.X > scrWidth {
		g.ball.pos.X, g.ball.pos.Y = scrWidth/2, scrHeight/2
		g.ball.velocity = &image.Point{X: rand.Intn(7-3+1) + 3, Y: rand.Intn(5-3+1) + 3}
	}
}
```

Let's talk about ball-to-player collision for a second. What do we want to happen when the ball hits the player's paddle? The ball should just reverse its X velocity and move in the opposite direction. What about the Y velocity? I feel like the Y velocity should be determined based on if the ball hit the top or bottom of the paddle. Let's take a look at the code.

```go
// ...
func (g *Game) checkBallCollision() {
	// ...

	// Player Collision
	//
	// If the ball's X position is less than or equal to the front of the paddle and the 
	// ball's Y position is between the top and center of the position of the paddle
	// we reverse the X velocity and add 1 to it. This is to increase the speed with each hit.
	// Also, set the Y velocity to it's negative velocity.
	// This means anytime the ball hits the top half of the paddle the ball will bounce
	// off in an upward direction.
	//
	// The reverse is true for the bottom part of the paddle.
	if g.ball.pos.X <= g.paddle.pos.X+g.paddle.img.Bounds().Dx()/2 &&
		g.ball.pos.Y >= g.paddle.pos.Y-g.paddle.img.Bounds().Dy()/2 &&
		g.ball.pos.Y <= g.paddle.pos.Y {
		g.ball.velocity.X *= -1
		g.ball.velocity.X += 1
		g.ball.velocity.Y = -int(math.Abs(float64(g.ball.velocity.Y)))
	}
	if g.ball.pos.X <= g.paddle.pos.X+g.paddle.img.Bounds().Dx()/2 &&
		g.ball.pos.Y <= g.paddle.pos.Y+g.paddle.img.Bounds().Dy()/2 &&
		g.ball.pos.Y >= g.paddle.pos.Y {
		g.ball.velocity.X *= -1
		g.ball.velocity.X += 1
		g.ball.velocity.Y = int(math.Abs(float64(g.ball.velocity.Y)))
	}
}
```

## The AI

We have a ball to send back and forth. We need a worthy opponent. Let's add the AI's movement logic. If the ball is above the position of the ai we want to move the AI up. If the ball is below the position of the AI we want the ai to move down. We also want to check bounds with the AI too. This isn't necessary though, the logic is that the ball will never move off screen so the AI shouldn't either, but safety first.

![](https://media.tenor.com/ry6gQ8oBbUgAAAAM/bts-kpop.gif)

```go

// ...

func (g *Game) aiMove() {
	if g.ball.pos.Y > g.ai.pos.Y {
		g.ai.pos.Y += g.ai.speedY
	}
	if g.ball.pos.Y < g.ai.pos.Y {
		g.ai.pos.Y += -g.ai.speedY
	}
}

// ...
func (g *Game) UpdatePlaying() {
	g.Move(g.paddle)
	g.checkPaddleBounds(g.paddle)
	g.ballMove()
	g.checkBallCollision()
	g.aiMove() // <-- Add this
	g.checkPaddleBounds(g.ai) // <-- Add this
}
// ...
```

## AI Collision

Now AI collision is the same as the player just different sides of the screen and paddle.

```go
// ...
func (g *Game) checkBallCollision() {
	// ...

	if g.ball.pos.X >= g.ai.pos.X-g.ai.img.Bounds().Dx()/2 &&
		g.ball.pos.Y >= g.ai.pos.Y-g.ai.img.Bounds().Dy()/2 &&
		g.ball.pos.Y <= g.ai.pos.Y {
		g.ball.velocity.X *= -1
		g.ball.velocity.X -= 1
		g.ball.velocity.Y = -int(math.Abs(float64(g.ball.velocity.Y)))
	}
	if g.ball.pos.X >= g.ai.pos.X-g.ai.img.Bounds().Dx()/2 &&
		g.ball.pos.Y <= g.ai.pos.Y+g.ai.img.Bounds().Dy()/2 &&
		g.ball.pos.Y >= g.ai.pos.Y {
		g.ball.velocity.X *= -1
		g.ball.velocity.X -= 1
		g.ball.velocity.Y = int(math.Abs(float64(g.ball.velocity.Y)))
	}
}
```

## Scoreboard

The gameplay is complete, sort of, we need polish. We need a scoreboard. 

To do that we are going to need text and when we need text we will need a font file and some initialization code. The font file, `Bodo Amat.tff`, I found it online for free. Thank you, font creators! 

```go
// ...

//go:embed "Bodo Amat.ttf"
var fontFile []byte

var gameFont font.Face

// ...

type Game struct {
	state       GameState
	bg          *ebiten.Image
	paddle      *Paddle
	ai          *Paddle
	ball        *Ball
	win         int
	loss        int
}

func init() {
	ff, err := truetype.Parse(fontFile)
	if err != nil {
		log.Fatal(err)
	}

	gameFont = truetype.NewFace(ff, &truetype.Options{
		Size:    24,
		DPI:     100,
		Hinting: font.HintingFull,
	})
}
```

That is it for the initialization of our font. Now we just need to draw our scoreboard.

```go
func (g *Game) DrawPlaying(scr *ebiten.Image) {
	g.bg.Fill(color.White)

	// ...

	winStr := fmt.Sprintf("Wins: %d", g.win)
	winBnds := text.BoundString(gameFont, winStr)
	dx, dy := 20, winBnds.Dy()/2+gameFont.Metrics().Ascent.Ceil()
	text.Draw(g.bg, winStr, gameFont, dx, dy, color.Black)

	lossStr := fmt.Sprintf("Losses: %d", g.loss)
	lossBnds := text.BoundString(gameFont, lossStr)
	dx, dy = scrWidth-lossBnds.Dx()-20, lossBnds.Dy()/2+gameFont.Metrics().Ascent.Ceil()
	text.Draw(g.bg, lossStr, gameFont, dx, dy, color.Black)

	scr.DrawImage(g.bg, nil)
}
```
For `winStr`, I set the `x` destination 20 pixels to the right. The `y` destination is half the height of the font + the ascent of the letters to ensure the text doesn't go off the top of the screen. 

With the `lossStr`, I perform the same actions, changing the `x` location.

## Updating the Scoreboard

Now we need to increment the value of the win or loss. We can do this in our `checkBallCollision()` method.

A win is when the ball's `x` position is greater than the AI's paddle position, and a loss is when the ball's `x` position is less than the player's paddle's `x` position.

```go
func (g *Game) checkBallCollision() {
	// ...
	// Edge collision.
	if g.ball.pos.X < 0 || g.ball.pos.X > scrWidth {
		if g.ball.pos.X < g.paddle.pos.X {
			g.loss++
		}
		if g.ball.pos.X > g.ai.pos.X {
			g.win++
		}

		g.ball.pos.X, g.ball.pos.Y = scrWidth/2, scrHeight/2
		g.ball.velocity = &image.Point{X: rand.Intn(7-3+1) + 3, Y: rand.Intn(5-3+1) + 3}
	}
}
```
When we test our game we should see wins and losses increment appropriately. 

## Main Menu

Finally, I think we hit the peak of our gameplay. We need to add a menu. Let's create a very basic menu nothing fancy. 

```go
// ...
func (g *Game) Draw(scr *ebiten.Image) {
	switch g.state {
	case Menu:
		g.DrawMenu(scr)
	case Playing:
		g.DrawPlaying(scr)
	case Quit:
		os.Exit(0)
	}
}

// ...
func (g *Game) DrawMenu(dst *ebiten.Image) {
	g.bg.Fill(color.White)

	menu := "Pong\n\n(P)lay\n(Q)uit"

	menuBnds := text.BoundString(gameFont, menu)
	dx, dy := scrWidth/2-menuBnds.Dx()/2, scrHeight/2-menuBnds.Dy()/2
	text.Draw(g.bg, menu, gameFont, dx, dy, color.Black)

	dst.DrawImage(g.bg, nil)
}
// ...
```

We need our menu to act appropriately when the `P` or `Q` button is pressed.

```go
// ...
func (g *Game) Update() error {
	switch g.state {
	case Menu:
		g.MenuUpdate()
	case Playing:
		g.UpdatePlaying()
	case Quit:
		os.Exit(0)
	}

	return nil
}

// ...
func (g *Game) MenuUpdate() {
	if ebiten.IsKeyPressed(ebiten.KeyQ) {
		os.Exit(0)
	}
	if ebiten.IsKeyPressed(ebiten.KeyP) {
		g.win = 0
		g.loss = 0
		g.ball.velocity = &image.Point{X: rand.Intn(7-3+1) + 3, Y: rand.Intn(6-3+1) + 3}
		g.state = Playing
	}
}
// ...
```

When we hit play we want to make sure the game resets itself.

## More Polish

That brings us to our next thing. What if we want to go back to the main menu when we are playing? We could just press the `X` on the game window and restart the game. I think we can do better though. 

```go
func (g *Game) UpdatePlaying() {
	// ...

	if ebiten.IsKeyPressed(ebiten.KeyEscape) {
		g.state = Menu
	}
}
```

## Audio 

It feels kind of bland, even for a pong clone. It just seems quiet. Let's add some sound. Three things should cause a sound. A ball hitting a paddle, a win, and a loss.

```go
// ...
//go:embed hitSound.wav
var hitFile []byte

//go:embed win.wav
var winFile []byte

//go:embed loss.wav
var lossFile []byte

// ...
type Stream int

const (
	HitSound Stream = iota
	Win
	Loss
)

// ...
type Game struct {
	// ...
	audioCtx    *audio.Context
	audioPlayer *audio.Player
	audioStream [][]byte
}

func main() {
	rand.Seed(time.Now().Unix())

	g := &Game{
		// ...
		audioCtx: audio.NewContext(48000),
	}

	g.audioStream = append(g.audioStream, hitFile)
	g.audioStream = append(g.audioStream, winFile)
	g.audioStream = append(g.audioStream, lossFile)

	ebiten.SetWindowTitle("Pong")
	if err := ebiten.RunGame(g); err != nil {
		log.Fatal(err)
	}
}
```

The `WAV` files I found online with free-to-use licenses. Thank you, sound creators! 

In the example above I embed the sound files into byte slices. I create an iota enumerated `Stream` type. This is to determine which sound to play and when. 

In the game struct, we add an audio context, a player and a slice of byte slices. The `audioStream` is to hold our sound files byte data.

Bringing all that together. We create a function to play audio for us.

> I don't know if this is an effiecient way to do this. We create a `NewPlayer` each played sound so if there are any suggestions on doing this better please open an issue or pull request on the repository's github.

```go
func (g *Game) playAudio(s Stream) {
	g.audioPlayer = g.audioCtx.NewPlayerFromBytes(g.audioStream[s])

	if err := g.audioPlayer.Rewind(); err != nil {
		log.Fatal(err)
	}
	g.audioPlayer.Play()
}
```

Now we can add sounds in the appropriate locations pretty easily.

```go
func (g *Game) checkBallCollision() {
	// ...
	if g.ball.pos.X < 0 || g.ball.pos.X > scrWidth {
		if g.ball.pos.X < g.paddle.pos.X {
			// ...
			g.playAudio(Loss)
		}
		if g.ball.pos.X > g.ai.pos.X {
			// ...
			g.playAudio(Win)
		}

		g.ball.pos.X, g.ball.pos.Y = scrWidth/2, scrHeight/2
		g.ball.velocity = &image.Point{X: rand.Intn(7-3+1) + 3, Y: rand.Intn(5-3+1) + 3}
	}

	// Player Collision
	if g.ball.pos.X <= g.paddle.pos.X+g.paddle.img.Bounds().Dx()/2 &&
		g.ball.pos.Y >= g.paddle.pos.Y-g.paddle.img.Bounds().Dy()/2 &&
		g.ball.pos.Y <= g.paddle.pos.Y {
		g.playAudio(HitSound)
		// ...
	}
	if g.ball.pos.X <= g.paddle.pos.X+g.paddle.img.Bounds().Dx()/2 &&
		g.ball.pos.Y <= g.paddle.pos.Y+g.paddle.img.Bounds().Dy()/2 &&
		g.ball.pos.Y >= g.paddle.pos.Y {
		g.playAudio(HitSound)
		// ...
	}

	// AI Collision
	if g.ball.pos.X >= g.ai.pos.X-g.ai.img.Bounds().Dx()/2 &&
		g.ball.pos.Y >= g.ai.pos.Y-g.ai.img.Bounds().Dy()/2 &&
		g.ball.pos.Y <= g.ai.pos.Y {
		g.playAudio(HitSound)
		// ...
	}
	if g.ball.pos.X >= g.ai.pos.X-g.ai.img.Bounds().Dx()/2 &&
		g.ball.pos.Y <= g.ai.pos.Y+g.ai.img.Bounds().Dy()/2 &&
		g.ball.pos.Y >= g.ai.pos.Y {
		g.playAudio(HitSound)
		// ...
	}
}
```

We could probably add some background music and stuff too but I'll leave that to you. 

## Extensions

I didn't do these but you can easily extend the gameplay here. 

* Make the game 2 players.
* Make the game 4 players. 
* Add powerups.
* Add attacks.
* Add shields from attacks.
* Add good sounds and music. 

## Final Word

Hopefully, someone found this post useful in some way. You can find the full source [here](https://github.com/KalebHawkins/pong). If there are any mistakes you can open an issue. If there are mistakes or corrections needed in the blog you can open an issue on the [blog's repository](https://github.com/KalebHawkins/KryoLogs).

#ebitengine #go