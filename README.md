# chess_puzzle_model
predicting the difficulty of a chess puzzle

Goal: 

My goal was to predict the ELO rating of a tactical puzzle on chess.com to within 100 points. 

Background: 

Chess.com users can practice their tactical abilities by training on puzzles. Both players and problems have ELO ratings. A player’s rating increases when they solve a puzzle correctly and decreases when they give an incorrect solution. A puzzle’s rating increases when a player gives an incorrect solution and decreases when they solve it correctly. This system allows chess.com to match players with puzzles of a difficulty fitting to their chess strength. Unfortunately, new puzzles do not have accurate ratings until they are given to many users, which means users may be presented with puzzles far above or far below their strength. 

Simple puzzles are easier to find than hard puzzles, so the data is heavily skewed to the right. The mean puzzle rating was 1067. Predicting the mean for each puzzle gives a baseline MSE of 705,240, which is a RMSE of 840. 


 

Data Processing: 

Chess.com was kind enough to provide me with 78,730 chess puzzles complete with ratings and solutions. I converted these into bitboards, binary matrices of shape. Similar to the way images are encoded, these bitboards have two spatial dimensions to represent the 64 squares on a chessboard and a depth dimension to represent different features that are expressed in the spatial dimensions. In a typical image, there are three channels representing red, blue, and green. A chess position can be expressed using 18 dimensions, 12 for each piece, 4 for kingside and queenside castling rights for white and black, one for whether en passant capture is legal, and one for who the active player is. For the first twelve, the bitboard is encoded as all 0s except for the square where a given piece is located. For the last 6, the entire channel is a 0 or a 1 depending on whether the condition is true or false. For example, the FEN  position '5rk1/pp4p1/2p1q1p1/4P1B1/2nn2P1/8/P4RK1/5R2 w - - 0 2' would look like this:  

White Pawn 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 1, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 1, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 1, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

White Rook 

[[ 0, 0, 0, 0, 0, 0, 0, 0], 

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 1, 0, 0],  

[ 0, 0, 0, 0, 0, 1, 0, 0]],  

 

White Bishop 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 1, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

White Knight 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

White Queen 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

White King 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 1, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black Pawn 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 1, 1, 0, 0, 0, 0, 1, 0],  

[ 0, 0, 1, 0, 0, 0, 1, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

 [ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black Rook 

[[ 0, 0, 0, 0, 0, 1, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black Bishop 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black Knight 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 1, 1, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black Queen 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 1, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black King 

[[ 0, 0, 0, 0, 0, 0, 1, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

White queenside castling legal? 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

White kingside castling legal? 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black queenside castling legal? 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

 [ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Black kingside castling legal? 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

En passant legal? 

[[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

 [ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0], 

 [ 0, 0, 0, 0, 0, 0, 0, 0],  

[ 0, 0, 0, 0, 0, 0, 0, 0]],  

 

Active player (0: black, 1: white 

[[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1],  

[ 1, 1, 1, 1, 1, 1, 1, 1]], 

 



 

 

 

 

In addition to the 18 FEN channels, I added 28 solution channels because the longest solution in my dataset was 28 moves. Each channel was all 0s except for a 1 where the piece started and a –1 where it ended. 

 

The last step in encoding my data involved calculating the Stockfish evaluation. Stockfish, the most powerful commercially available chess engine, expresses an evaluation in terms of centipawns until it can calculate all the way to checkmate, at which point it switches to ‘Mate in X’. I added 10 channels to correspond to the following ten evaluation ranges: 

< .2 cp                  Mate in 1 

.2 - 2 cp                Mate in 2 

2 - 4 cp                 Mate in 3 

4 - 7 cp                 Mate in 4 

6+ cp                    Mate in 5+ 

Nine of these channels remained all 0s while the channel corresponding to the appropriate evaluation was changed to all 1s. 

Convolutional Neural Network: 

A chessboard can be treated as a picture with 64 pixels and 56 colors, so it made sense for me to adopt some techniques used in image processing models. A convolutional neural network helped this model learn the spatial relationships among the pieces. I used 3x3 convolutions with same padding in order to preserve the shape of the chessboard. I also applied l2 regularization in each convolutional layer to reduce overfitting. Because the model needs to learn relationships between pieces on opposite sides of the board, it is necessary to include at least 7 convolutional layers. My model used 16, which I divided into 8 residual blocks. 

Residual blocks: 

The vanishing gradient problem is a persistent problem in deep neural networks. Without getting too technical, the problem is that the model computes its gradients using the chain rule, which involves multiplication by previous gradients. Since these gradients are all less than 1, the gradients eventually become so small that the model ceases learning. One way of solving this is using a residual neural network. In my model, I implemented skip connections every two layers. Basically, the model passes an input tensor through 2 convolutional layers and then adds the resultant tensor to the input tensor. This preserves information from the original tensor but also factors in the new information that was just learned. If the input tensor has fewer channels than the output tensor, I pass it through a 1x1 convolutional layer before adding input and output tensors. 

Each convolutional layer in the residual block really consists of four parts. The first part is the Conv2D layer, in which I apply the convolutions. Next, is a batch normalization layer, which is another way to combat the vanishing gradient problem. Batch normalization normalizes the weights of the network so that they never get too small to respond to adjustments. Nest, I apply a relu activation function. Lastly, I apply a Dropout layer of .5 to further reduce overfitting. 

Finally, I apply squeeze-excitation to apply weighting to each channel. Normally, each channel in a tensor is given equal weights, but in a chess position some of the channels are more important than others, and squeeze-excitation helps these relationships express themselves. 

Model architecture: 

My model uses 8 residual blocks with filters of 32, 32, 64, 64, 64, 64, 128, 128. Increasing the number of filters allows the model to remember more complex relationships as the it trains. After the convolutional layers, I apply a layer with 4 filters to reduce the complexity before flattening. Then, I flatten the tensor and pass it through a dense layer with 512 filters. My code can be viewed in the notebook in this repo. 

 

Results: 

My final model had a MSE of 249938.0. In short, I was able to predict the difficulty of these models to within 500 ELO points. The model is still demonstrating overfitting despite my efforts to regularize, and the test MSE stabilizes after just a few epochs.  

 

 

 
 
