# CV
### Contact Info
Ana Skalabava<br>
**E-mail:** pamjaranka@gmail.com.<br>
**Mobile:** +375297645943.<br>
**Linkedin:** https://www.linkedin.com/in/ana-skalabava-725b1ba6/.
### Summary
My main work experience concerns front-end development for quick termed projects. Plus back-end practice with PHP/WordPress.
I want to expand my professional knowledge base, try to work in an experienced team in a big company with established work processes.
### Skills
4+ years of experience development of the UI/UX interactive front-end, using HTML, JavaScript(ES5-ES6), React, CSS/SCSS.<br>
Experience with server–side modules development using PHP.<br>
Experience with source control system Git.<br>
Experience with a structured build, including Gulp, Webpack, Grunt and NPM.<br>
IDE: SublimeText, PhpStorm.
### Code examples (React Component)
```import React, {Component} from 'react';
import Draggable from 'react-draggable';
import {connect} from 'react-redux';
import {
    boardBoundsSelector,
    activeBoardIDSelector,
    optionPositionSelector,
    boardBoundsIsLoadedSelector,
    participantCodeSelector,
    endeavorIdSelector,
    colorCode
} from '../../selectors';
import {OPTION_BOX_SHADOW} from '../../constants/styled';
import {setOptionPosition, postOptionPosition} from '../../ac'
import Loader from "../Loader";
import {NO_DATA} from '../../constants/actions';
import ReactDom from "react-dom";
import Option from '../../components/Option';

class Sticker extends Component {
    state = {
        isDraggable: this.props.isDraggable === 'true' || false,
        color: colorCode(this.props.sticker.color),
        positionWasChanged: true
    };

    constructor(props) {
        super(props);
        this.handleStop = this.handleStop.bind(this);
        this.handleDrag = this.handleDrag.bind(this);
        this.myRef = React.createRef();
        this.optionRef = React.createRef();
    };

    componentDidUpdate(prevProps, prevState, snapshot) {
        if(this.myRefBounds) return;

        const componentBounds = ReactDom.findDOMNode(this.myRef.current).getBoundingClientRect();
        const left = componentBounds.left - this.props.boardBounds.wrapperLeft;
        const top = componentBounds.top - this.props.boardBounds.wrapperTop;

        this.myRefBounds = {
            left: left,
            top: top,
            right: left + componentBounds.width,
            bottom: top + componentBounds.height,
        };

        this.optionNode = ReactDom.findDOMNode(this.optionRef.current);
    }

    render() {
        return (
            <div>
                {!this.props.boardBoundsIsLoaded ?
                    <Loader/> :
                    this.body
                }
            </div>
        );
    }

    get body() {
        const position = this.getDeltaPosition();

        let dragHandlers = {
            onStop: this.onStop,
            onDrag: this.onDrag,
            handleStop: this.handleStop,
            handleDrag: this.handleDrag,
        };

        if(position.x >= 0 && position.y >= 0) {
            dragHandlers = {...dragHandlers,  position: {x: position.x, y: position.y}};
        }

        if (!this.state.isDraggable) {
            dragHandlers = {...dragHandlers, disabled: true};
        }

        return (
            <div ref={this.myRef}>
            <Draggable {...dragHandlers}>
                <div><Option option={this.props.sticker} ref={this.optionRef}/></div>
            </Draggable>
            </div>
        )
    }

    getBoardBounds() {
        return {
            bottom: this.props.boardBounds.get('bottom'),
            left: this.props.boardBounds.get('left'),
            right: this.props.boardBounds.get('right'),
            top: this.props.boardBounds.get('top')
        }
    }

    onStop(e, data) {
        this.handleStop(data, this.defaultPosition);
    }

    handleStop(data, defaultPosition) {
        this.setPositions(data, defaultPosition);
        this.setDefaultStyle(data.node);
    }

    onDrag(e, data) {
        this.handleDrag(data);
    }

    handleDrag(data) {
        if(this.checkPosition(data)) {
            this.setValidActiveStyle(data.node);
        } else {
            this.setDefaultStyle(data.node);
        }
    }

    setPositions(data, defaultPosition) {
        const isPositionValid = this.checkPosition(data);
        const absolutePositions = this.getAbsolutePosition(data);

        const positionX = isPositionValid ? absolutePositions.x : defaultPosition.x;
        const positionY = isPositionValid ? absolutePositions.y : defaultPosition.y;

        if((positionX !== this.props.position.positionX) ||
            (positionY !== this.props.position.positionY)) {

             this.props.setOptionPosition({
                id: this.props.activeBoardId,
                option: {
                    id: this.props.sticker.id,
                    positionX: positionX,
                    positionY: positionY,
                }
            });

             this.props.postOptionPosition({
                 endeavor_code: this.props.endeavorId,
                 participant_code: this.props.participantCode,
                 position_x: positionX,
                 position_y: positionY,
                 option_id: this.props.sticker.id
             });

            this.setState({
                'positionWasChanged': true
            })
        }
    }

    getAbsolutePosition(data) {
        const bounds = this.getBoardBounds();
        const absoluteX = this.myRefBounds.left + data.x - bounds.left;
        const absoluteY = this.myRefBounds.top + data.y - bounds.top;

        return {
            x: absoluteX,
            y: absoluteY,
        }
    }

    getDeltaPosition() {
        const {positionX, positionY} = this.props.position;

        if(positionX === NO_DATA || positionY === NO_DATA) {
            return {
                x: 0,
                y: 0,
            }
        }

        const bounds = this.getBoardBounds();

        const deltaX = bounds.left + positionX - this.myRefBounds.left;
        const deltaY = bounds.top + positionY - this.myRefBounds.top;

        return {
            x: deltaX,
            y: deltaY,
        }
    }

    checkPosition(data) {
        const bounds = this.getBoardBounds();
        
        return bounds.top < this.myRefBounds.top + data.y &&
            bounds.left < this.myRefBounds.left + data.x &&
            bounds.bottom > this.myRefBounds.bottom + data.y &&
            bounds.right > this.myRefBounds.right + data.x
    }

    setValidActiveStyle(node) {
        this.optionNode.style.boxShadow = `0px 0px 14px 2px ${this.state.color}`;
        this.optionNode.style.border = '1px solid rgba(256,256,268,0.4)';
    }

    setDefaultStyle(node) {
        this.optionNode.style.boxShadow = OPTION_BOX_SHADOW;
        this.optionNode.style.border = `1px solid ${this.state.color}`;
    }
}

const initMapStateToProps = () => {
    const positionSelector = optionPositionSelector();
    return (store, ownProps) => {
        return {
            boardBounds: boardBoundsSelector(store),
            boardBoundsIsLoaded: boardBoundsIsLoadedSelector(store),
            activeBoardId: activeBoardIDSelector(store),
            position: positionSelector(store, ownProps.sticker),
            participantCode: participantCodeSelector(store),
            endeavorId: endeavorIdSelector(store)
        }
    }
}

export default connect(
    initMapStateToProps,
    {
        setOptionPosition: setOptionPosition,
        postOptionPosition: postOptionPosition
    }
)(Sticker)
```
### Experience
##### 2015 - 2019
Dizzain Inc.<br>
Team Lead, Front-end Developer<br>
Managing and growing staff team.<br>
Responsive, interactive front-end design.<br>
Architecting and writing code (JavaScript, CSS/SCSS, HTML).<br>
Wordpress theme development, including designing PHP modules.<br>
React App development.<br>
Interviewing prospective employees.
##### 2015
7Version<br>
Front-end development using jQuery, HTML, CSS, including CSS framework Bootstrap.<br>
Responsive, adaptive design.<br>
Wordpress theme development using PHP.
##### 2014
CIM<br>
Front-end development using HTML, CSS and jQuery.<br>
Responsive, adaptive design.<br>
Template development using PHP and web template system Smarty.<br>
Experience with using PuTTY/KiTTY.
### Education
##### 2005 – 2010
Belarusian State Tehnological University<br>
Faculty of Printing and Publishing<br>
##### 2017
Course "JavaScript/​DOM/Interfaces" on https://learn.javascript.ru/.
##### 2019
Course "React.JS" on https://learn.javascript.ru/.
### English
My English level is Upper-Intermediate. Professional experience includes communication with customers. Plus spoken practice with foreigners abroad or upcountry.