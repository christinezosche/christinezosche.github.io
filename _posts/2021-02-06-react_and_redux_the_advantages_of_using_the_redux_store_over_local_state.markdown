---
layout: post
title:      "React and Redux: The advantages of using the Redux store over local state"
date:       2021-02-07 00:25:48 +0000
permalink:  react_and_redux_the_advantages_of_using_the_redux_store_over_local_state
---

Inspired by the easy dynamic rendering that React provides, I decided that for this build, I wanted to challenge myself to create a game. Not a complex one, at least not for my first try at making a game. I settled on memory, like the classic game of flash cards. Simple, right? Think again…

Even a simple game like this one has many moving parts. A timer, game cards of differing varieties, turns, and of course, the logic of what to do when a pair of cards match. Luckily, React provides a way to manage data for each component through component state. 

When I started to build the game, I focused on managing the logic using component state:

```
import React, { Component } from 'react'
import GameCard from './GameCard'

class GameCards extends Component {

    constructor() {
        super()

        this.state = {
            currentPair: [],
            completedPairs: [],
            newTurn: false,
            gameCompleted: false
        }
    }

    addToCurrentPair = (imageId) => {
        this.setState({
            currentPair: [...this.state.currentPair, imageId]
        })
    }

    hasBeenMatched = (imageId) => {
        if (this.state.completedPairs.filter(id => id === imageId).length === 0) {
            return false
        }
        else {
            return true
        }
    }

    checkForPairs = () => {

        if (this.state.currentPair.length === 2) {
            if (this.state.currentPair[0] === this.state.currentPair[1]) {
                this.setState({
                    currentPair: [],
                    completedPairs: [...this.state.completedPairs, this.state.currentPair[0]],
                    newTurn: true
                })
            }
            else {
                this.setState({
                    currentPair: [],
                    newTurn: true
                })
            }
        }
        else if (this.state.newTurn === false) {
        }
        else {
            this.setState({
                newTurn: false
            })
        }
    this.checkForGameOver()
    }

    checkForGameOver = () => {
        if (this.state.completedPairs.length === 6) {
            setTimeout(() => { this.setState({
                gameCompleted: true
            })}, 1000);
            this.props.setDelay()
            this.props.dispatchGame()
        }
    }

    renderGame = () => {
        if (this.state.gameCompleted === true) {
            return <div className="game-container">{this.renderEnd()}</div>        
        }
        else {
            return <div className="game-container">{this.renderCards()}</div>
        }
    }

    renderEnd = () => {
        return <h1>Game Over!</h1>
    }

    renderCards = () => {
        return this.props.images.map((image) => <GameCard imageUrl={image.url} imageId={image.id} addToCurrentPair={this.addToCurrentPair} checkForPairs={this.checkForPairs} hasBeenMatched={this.hasBeenMatched} newTurn={this.state.newTurn} />)
    }

    render () {
        return (
            <div>{this.renderGame()}</div>
        )
    }
}

export default GameCards
```

Already a lot of moving parts, and this is only ONE component. It worked, but not for long…

In order for each component to render properly on each turn, the state of each component needed to be updated after each move, depending on what happened: whether two cards were clicked, whether the cards were a match, and whether the game was over. With several components interacting at once, that's a lot of props being passed around among components, and more importantly, a lot of setState. Making sure that each component received the right information was hard enough, but I probably could have managed it. The issues got much worse due to a specific feature of React: components re-render upon changes in state. Additionally, state updates are asynchronous, so the information stored in state is not necessarily immediately available to a component. This created many infinite loops, endless re-rendering, among several components. While React has many lifecycle methods available to mitigate these problems, there is a cleaner, better, simpler solution:

Redux. By managing data in the Redux store, every component in the app has access to the exact same set of data, in real time. Thus, upon a change in the state of the Redux store, each component can render on its own, without the need for props to be passed around among components and state to be set. Much less room for error, much less code, and no infinite loops!

```
import React, { Component } from 'react'
import GameCard from './GameCard'
import { connect } from 'react-redux';
import { setGameComplete, clearCurrentPair, addToCompletedPairs, clearHoldImages, setNewTurn, setPostClickDelay } from '../actions/games'

class GameCards extends Component {

    checkForPairs = () => {

        if (this.props.currentPair.length === 2) {
            if (this.props.currentPair[0].imageId === this.props.currentPair[1].imageId) {
                this.props.addToCompletedPairs(this.props.currentPair[0].id)
                this.props.addToCompletedPairs(this.props.currentPair[1].id)
                this.props.clearCurrentPair()
                setTimeout(() => { this.props.clearHoldImages() }, 1500); 
                this.props.setNewTurn(true)
                this.props.setPostClickDelay(true)        
                setTimeout(() => { this.props.setPostClickDelay(false) }, 1500); 
            }
            else {
                this.props.clearCurrentPair()
                setTimeout(() => { this.props.clearHoldImages() }, 1500); 
                this.props.setNewTurn(true)
                this.props.setPostClickDelay(true)        
                setTimeout(() => { this.props.setPostClickDelay(false) }, 1500); 
            }
        }
        else {
            this.props.setNewTurn(false)
        }
    this.checkForGameOver()
    }

    checkForGameOver = () => {
        if (this.props.completedPairs.length === 24) {
            setTimeout(() => { this.props.setGameComplete() }, 1000)
        }
    }

    renderGame = () => {
        // if (this.props.gameComplete === true) {
        //     return <div className="game-container">{this.renderEnd()}</div>        
        // }
        // else {
            return <div className="game-container">{this.renderCards()}</div>
        // }
    }

    // renderEnd = () => {
    //     return (
    //     <div>
    //     <h1>Game Over!</h1>
    //     <h2>Your Time: {this.props.time}</h2>
    //     </div>
    //     )
    // }

    renderCards = () => {
        return this.props.images.map((image) => <GameCard key={image.id} imageUrl={image.url} imageId={image.imageId} />)
    }

    componentDidUpdate = () => {
        this.checkForPairs()
    }

    render () {
        return (
            <div>{this.renderGame()}</div>
        )
    }
}

…

```

Making sure the game worked was hard enough. Without the Redux store, I may have been stuck in an infinite loop myself.

