# Seq2Event: Learning the Language of Soccer using

<div style="text-align: center; margin: 30px 0; padding: 20px; background-color: #f9f9f9; border-radius: 8px;">
    <p style="color: #333; font-weight: 500;">Transformer-based Match Event Prediction, IanSimpson RyanJ.Beal, Southampton,UK Southampton,UK, ijs1c20@soton.ac.uk ryan.beal@soton.ac.uk</p>
    <p style="color: #666;">KDD 2022</p>
    <p style="margin-top: 12px;">
        📄 <a href="https://dl.acm.org/doi/abs/10.1145/3534678.3539138" target="_blank" style="color: #1565c0;">论文原文</a>
    </p>
</div>

## 2 Background

ratherthanafullsequence,thedecoderstackisnotrequired,and
Inthissection,literatureontechniquesformodellingsequencesis thereforeonlytheTransformerencoderstackisemployed.
reviewed,followedbyasummaryofmetricsusedtoidentifyteam Historically,theuseofLSTMandTransformermodelsinsoccer
strategiesinsoccer. hasfocusedonformatchvideoprocessingtaskssuchasannotation
[30]andsummarisation[2],althoughrecentlyotherauthorshave
2.1 ModellingSequences alsobeguntolookatteambehaviourprediction[15,35].
Traditionalstatisticalarereviewedinordertoformbaselinemodels,
andmachinelearningtechniquesusedintheSeq2Eventmodelsare
2.2 MetricsforIdentifyingTeamStrategiesin
reviewed.
BaselineTechniques.AutoRegressive(AR)modelsareone Soccer
componentofthepopularAutoRegressiveIntegratedMovingAver- ExpectedGoals(xG)wasoriginallyintroducedtothesportofice
age(ARIMA)familyofmodels[16],andserveasasimplebaseline hockey[21].Itisidentifiedthatalimitationoficehockeyisthe

Seq2Event:LearningtheLanguageofSoccerusingTransformer-basedMatchEventPrediction 
lowscoringratecomparedtoothersportssuchasbasketball.The CrossEntropyLoss(CEL)ispopularlyusedforequivalentmulti-
paperidentifiesthattherandomnessandscarcityofgoalslimits classclassificationtasksinNLP,andisusedheretomeasureerrorof
theabilitytoproperlyjudgecurrentperformanceandtopredict thepredictionofthenextaction.Root-Mean-Square-Error(RMSE)
futureperformanceofteamsusinggoalsalone.Sparsityofscoring losswasidentifiedassuitableforthe𝑥,𝑦co-ordinateerror,particu-
isadifficultysharedwithsoccer[9].Inordertoprovideamore larlyduetothismetric’sintuitionasEuclideandistanceonaspatial
continuouscontext,thexGmodelallowsanestimateofthenumber metric.Applyinganunweightedsumwasfoundtoleadtoabias
ofgoalsateamoughttohavescoredbasedontheirperformance towardsmodelsthatprioritisedreducingtheRMSEloss,andsothe
onkeymetricssuchasshotsongoal,missedshots,blockedshots, finallossfunctionispresentedasaweightedsumwiththeweights
turnovers,face-offs,andactualgoalsscored.OrdinaryLeastSquares determinedempirically.Theresultantlossfunctionisshownin
(OLS)regressionandRidgeregressionwereusedonatime-series Equation1.
ofthesemetricstobuildmodelsbyxG[21].xGwasfirstrecorded
insoccerin2016[12],withmodelsbasedonrandomforestand
L(𝑦ˆ,𝑦)=5·𝐶𝐸𝐿(𝑦ˆ 𝑎𝑐𝑡𝑖𝑜𝑛,𝑦 𝑎𝑐𝑡𝑖𝑜𝑛)+𝑅𝑀𝑆𝐸(𝑦ˆ 𝑙𝑜𝑐,𝑦 𝑙𝑜𝑐) (1)
Adaboost[12]. Eventactiontypehasaclassimbalance,sotoresolvethis,the
AlimitationofxGisthatitisafunctionofinstantaneousshot CELisitselfweightedonthereciprocalclassoccurrenceproportion.
actionsonly.Itdoesnotaccountforeventsthatdeliveropportu- Thismeans,forexample,thatthestrongtruepositiveofa‘pass’
nitiesthatdonotresultinsignificantmatcheventsfromwhich willnotyieldaslowalossasanequivalentlystrongtruepositive
xGcouldbeaccrued.Severalmetricsaimtoaccountforthis,such ofa‘shot’,sinceshotsoccurlessfrequently(1.4%vs56%).Scoring
asthe Off-BallScoringOpportunity[31],ExpectedAssists (xA) goals,changeofpossessionevents,andend-of-matcheventsare
[23],ExpectedGoalChain(xGChain)[20],ExpectedGoalBuild-Up purelycontextualandarenotconsideredindicatorsofstyleforthis
(xGBuildup)[29],ExpectedThreat(xT)[29]metrics. task,soaregivenzeroweightswithintheCELfunction.
AmoreholisticapproachwastakenbytheValuingActionsby
EstimatingProbabilities(VAEP)metric[10],whichisstillonlya 3.2 Seq2EventModelArchitecture
functionofon-ballactions,butestimatestheprobabilityofscoring TheSeq2Eventmodelcomprisessevenmainstages,withRNNand
agoalinthenearfutureacrossallactions.Themetrichasproven Transformervariants,asshowninFigure1.Inputsarematchevents
successfulandsuccinctlyaccountsforoverandunderachievement from𝑡−𝑠𝑒𝑞𝑙𝑒𝑛to𝑡−1,withoutputanestimateoftheeventat𝑡.
inbothscoringandconcedingaspectsofthegame.Production
ofVAEPscoresisunderpinnedbytheneedforamodelforthe Stage1:LearnableEmbedding.Forthistask,wecouldsimply
scoringandconcedingprobabilities,forwhichDecroosetal.[10] treattheactionsaswords.However,weknowthatthecontextofan
useaCatboostgradientboostedmodelonthepastthreeactionsto eventhasmeaningandwishtocapturethetencontinuousvariables
generateprobabilitiesforwhetheragoalwasscoredorconceded inthemodel.Anembeddinglayerisusedtoembedtheactionsand
next. adenselayerisusedtotransformthecontinuousvariables.
EstimatingtherewardofteamtacticsisthefocusofBealetal. Hyperparameters:
[3],and,buildingonthis,thelongtermoptimisationofdecision
• seqlen:sequencelength;howfarbackthemodellooks.
making[4].‘Fluentobjectives’aredefinedforateam,whichare
• act_embed_dim:actionembeddingdimensionality.
afunctionof‘objectivevariables’whichcorrespondtodifferent
• cont_embed_dim:continuousfeaturedenselayerdimension-
pointsintheplanninghorizonofanagent.MarkovChainMonte
ality.
Carlo(MCMC)isperformedinordertomakepredictionsabout
thefutureperformanceofstrategiesagainstdifferentobjectives, Stage2:Concatenation.Weneedtopassonematrix,composed
andthisisusedtoinformtheselectionofoptimalobjectivesbythe ofvectorsofsequentialeventstotheRNN/Transformer,andsotwo
agent.SeeBealetal.[5]foramorecompletereview. outputsfromthepreviousstageareconcatenated.IntheTrans-
formervariant,positionalembeddingisalsoappliedatthisstage.
3 SEQ2EVENTMODELFORMATCHEVENT
Stage3:RNN/TransformerComponent.EitheranRNNora
Transformerencodercomponentoperatesinthisstage.
PREDICTION
RNNvarianthyperparameters:
Inthissectionthearchitecture,andassociatedlossfunction,ofour
• RNNtype:oneofElman-RNN,LSTM,orGRUisspecified.
novelSeq2Eventmodelformatcheventpredictionarepresented.
• hiddensize:thenumberofhiddenunitswithineachRNN
cell.
3.1 LossFunction • numberoflayers:thenumberofstackedRNNcells.
Definingalossfunctionthatcharacterisestheerrorisnecessary • dropoutrate:ifnumberoflayers>1,theproportionoffinal
tocorrectlymeasurethesuccessofamodel.Whilstallofthefea- layerweightsstochasticallyignoredduringtraining.
turesinthedatasetarerelevantasmodelinputs,notallprovide • directionality:unidirectional:thisstageisafunctionofor-
differentationofstylesofplay.Forexample,scoreadvantageisan deredinputsfromstarttoendofinputsequence;elsebidi-
importantcontextualinput,butchangesinfrequentlyandisthere- rectional:thisstageisafunctionoforderedinputsfromstart
forenotofinteresttodirectlypredict.Actiontypeandresultant toend,thenendtostartofinputsequence.
spatiallocationin𝑥,𝑦ofon-the-ballactionswerechosenastarget • activationfunction:forElmanRNN,theactivationfunction
variables,astheywerethemostfundamentalfeaturesofinterest, (e.g.ReLU)appliedpriortoupdatingthehiddenstateateach
andaredirectlyobservedandrecordedinthesourcedata. timeiteration.

 IanSimpson,RyanJ.Beal,DuncanLocke,andTimothyJ.Norman
Figure1:Seq2Eventmodelarchitecture.Predictionsofnextactionandpositionaremadegivenasourceof11×𝑠𝑒𝑞𝑙𝑒𝑛comprising
tencontinuousfeaturesandonecategoricalactionfeature.Learnablecomponentsareshadedorangeandred.
Transformervarianthyperparameters: totheseoutputs.Forpracticalapplication,simplifiedfeatureengi-
neeredactionspass(‘p’),dribble(‘d’),cross(‘x’),andshot(‘s’)were
• feedforwarddimension:analogoustotheRNNhiddensize. used,aselaborateduponinAppendixA.Thesoftmaxofthefour
• number of heads: the number of heads in the multi-head actionlogitsistaken,yieldingnextactionpredictionprobabilities.
attentioncomponent.
Eachindividualpredictionismadegiventheprevious40events
• numberoflayers:thenumberofparallellayersinthestack. intheexampleofmodeloutputsshowninFigure2.Gapsinthe𝑥,𝑦
• dropoutrate:analogoustotheRNNdropoutrate. plotsareperiodsofplaywheretheteamisnotinpossession,which
areshownas‘_’intheactionsequence.Themodeldoesnotpredict
Stage4:Extractionoffinalprediction.TheRNN/Transformer
turnoversastheyaregivenzeroweightinthelossfunction,which
componentmakesone-aheadpredictionsacrossthewholeset.How-
isbydesign,sincetheintendedpurposeistoanalyseattacking
ever,forthistaskweareonlyinterestedinthefinalpredictionof
behaviourwhereturnoverpredictionisoflessrelevance.However
theset,representingthenextevent.
thelossfunctionand/orthefinalstageofthemodelcouldeasily
Stage5:DenseLayerwithReLUActivation.Thefocusfrom
bemodifiedtooperateonothertasksthatcouldincludeturnover
Stage5onwardsisonmappingtherepresentationtoactionand
prediction.
𝑥,𝑦 outputs.Adenselayerwithrectifiedlinearunit(ReLU)[22]
activationfunctionisapplied.Initialprototypesexcludedthislayer,
butJavidetal.[18]refertothepotentialforsuchastructureto
aidwithlearningparticularlywhenlargerhiddenunitsizesare
used,sinceneuronoutputsmaybeignoredwhentheyarebelow
the𝑥 = 0activationthreshold.Applicationtoprototypemodels
showedamodestimprovementintrainingloss(circa1%).
Hyperparameter:
• outputdimensionality:denselayerdimensionality.
Stage6:DenseLayer.Thefinallearnablelayerisadenselayer
mappingtherepresentationtothefinaloutputs.Sevenactiontypes
arepredicted,althoughthreeofthesetypesareredundantsincedue Actualactions: ppdpdppppppxsxp_ ppp_ pdp_ pdp_ ppppppppppp
Predictedactions:ppppxpppppddssxx dpdx dpdp pdds pdpppdddppp
tobeinggivenzeroweightinginthelossfunction,andtherefore Eventsequence: 201.. ..215224..226 229..231 234..236 240.. ..250
couldbereducedtofouractiontypesinfutureversions.
Stage7:Splitting.Thevectoroflength9issplitintoavectorof Figure2:Seq2Eventmodelpredictions(blue)ofnexteventlo-
actionlogitsoflength7(forthefouractionsplusthreechangeof cationandactionvsactual(black),givencontextofprevious
possessioncharacters),andavectorof𝑥,𝑦positionco-ordinatesof 40eventsateachtimestep.Gapsinlocation(grey)indicate
length2.Undertraining,Equation1isappliedasthelossfunction possessionturnovers.

Seq2Event:LearningtheLanguageofSoccerusingTransformer-basedMatchEventPrediction 
4 EMPIRICALEVALUATION models,includingLSTM,mustoperateinseriesonthedata,with
Toevaluateourmodel,matcheventdatafromtheWyScoutOpen computationalcomplexityoforderlineartothesourcesequence
AccessDataset[24]wasused,coveringthe2017/18seasonofthe length[26].
English,French,German,ItalianandSpanishmen’sfirstdivisions, LSTMmodelsyieldedthebestresultsbytestloss,andoutper-
plustheinternationalEuro2016andWorldCup2018competitions. formedtheTransformermodelwithequivalentconfiguration(test
Comprising1,941matchesintotal,asampleof138matchesrepre- loss0.332vs0.379),demonstratinganenhancedabilitytolearn
sentativeacrosssuccessandcompetitionswasused.Furtherdetails patternsinlongersequenceswhichhasalsobeennotedwiththis
onthereproducibilityofourresultscanbefoundinAppendixA. architectureinotherdomains[7,8].
GRUmodelshavefewerlearnableparametersthanLSTMand
thisresultedinamarginallyfastertrainingtime,buttestperfor-
4.1 Experiment1:HyperparameterSelection mancewasnotasgoodasLSTMorTransformermodels.Elman-
Hyperparametersearchwasconductedbyinitialexperimentation RNNmodelsaretheearliesttypeofRNNandgenerallyprovided
acrossawiderangeofvaluesonallparameterstoidentifyplau- theworsttestperformanceofallSeq2Eventvariants.
siblyoptimalvaluesonwhichhyperparametergridsearchwas Models with sequence lengths of 5, RNN models with more
conducted.Atotalof145differentmodelswerefitted,usingdata than2layers,andTransformermodelswithmorethan2headsall
streamedasshowninFigure3.Allmodels’performancebytest performedpoorly(bestmodellosses0.423,0.440,0.384respectively).
lossisshowninFigure4. WehighlightLSTMasbeingpromisingforfutureresearches-
Continuousfeatureswereengineeredfromon-the-ball𝑥,𝑦,and pecially involving sequence lengths of 100 or more, where the
timedatatoprovidefeaturesofknowncontextualsignificancein computationalresourcescanbejustified.Fortheapplicationof
socceranalytics[1,10]whilstbeingsomewhatindependentinrep- identifyingteamstrategiesbyexaminingdifferencesagainstwhat
resentation.Theresultantsetcomprised𝑥,𝑦,𝑇,andtheirrespective ‘theaverageteam’woulddo,lookingbackattoomanyactionscan
deltabetweenobservations;distancecoveredbetweenobservations; beundesirable.Themodelmaypickupthe‘style’oftheteamand
angleanddistancetooppositiongoal;andscoreadvantage(with starttobenefitbypredictingthenexteventusingthat,providing
negativevaluesindicatingagoaldeficit).Acategoricalactionfea- predictionsagainstwhat‘thecurrentteam’woulddo.Inthenext
turewasengineeredbymappingthe106possibleactiontypesonto sectionsofthispaper,theTransformermodelwiththethirdbest
fourgeneralisedactiontypes:pass,dribble,cross,andshoot.Reduc- performanceandsequencelengthof40wasusedtogenerateresults
tionwasmadeaccordingtothreeprinciples:(1)categorysupport duetoitssuperiorspeedofcomputation.
hadtobesufficientsoastopermitlearningonthemodestlysized
dataset;(2)actionshadtobesufficientlydistinctintheirmeaning
inthecontextofthesoccerdomain;(3)thesimplifiedencodinghad
toyieldsequencesofreasonablesequentialdiversity.
Withthetaskofpredictingthenextoffensiveactiongivena
team’s previous offensive actions, only the offensive actions of
theteamofinterestwerekeptpermatch,butwithachangeof
possessioncharacteradded.Thus,observationsofaspecifiedlength
andstepcanbemadeacrossthedata,asshowninFigure3.
Wefitted12baselineAutoregressiveandMarkovchainmodels
of orders 1-5 and on𝑥,𝑦 and all ten continuous variables. The
bestbaselinemodelwasoforder1onalltencontinuousvariables
(testloss0.704).TheworstSeq2Eventmodelbetteredthisscore,a Figure3:WyScoutdataisrearrangedtoformstreamsfor
Transformervariantwith17headsandahiddenunitsize4096(test Seq2Eventmodelling.Fromsourcedataorderedontime(top
loss0.548).ThebestperformingwasanLSTMvariantwithsequence tworows),periodsofpossessionforeachteamareidentified
length100,1layer,hiddenunitsize8,andunidirectionalorder(test (thirdrow).Eventsfromeachpossessionareaggregatedby
loss0.332),withitsbidirectionalequivalentcomingsecond(test teamwithaturnoverindicatoraddedbetweenpossessions.
loss0.344).ATransformervariantwithsequencelength40,1layer,
andhiddenunitsize8camethird(testloss0.362).
APyTorchimplementationofthemodel[28]wasrunonGoogle
ColabProhardware,withaccelerationprovidedbyanNvidiaTesla
P100GPU.Ingeneral,forequivalentsettings,Transformermodels
werequickertotrainthanRNNvariants.ThebestLSTMmodel
took15.5htotrain.Reducingthesequencelengthto40reducedthe
modeltrainingtimetobetween3.5to5.5hdependingonotherhy-
perparameters,butthiswasstillmuchlongerthanthecomparable
bestTransformermodelwhichtook1.4htotrain.Thisvalidates
oneoftheknownbenefitsofTransformermodels,inthatthear- Figure4:Train(blue),validation(yellow)andtest(black)loss
chitectureavoidsiteratingoverthedata,thusspeedingupfitting overallmodelstrainedduringhyperparametersearch.
aslongasthewholesourcedatafitsintomemory;whereasRNN

 IanSimpson,RyanJ.Beal,DuncanLocke,andTimothyJ.Norman
4.2 Experiment2:AnalysisofActionPrediction Table1:MulticlassConfusionMatrix(MeanProbability)
Probability
Actualoccurrenceofactionswasassessedagainstthepredicted PredictedAction
ActualAction Pass Dribble Cross Shot
probability of action occurrence, for all four action types (pass,
dribble,cross,shot).Focusinghereonshotprediction,Figure5 Pass 0.36 0.38 0.16 0.10
showsthespatialdistributionofrelevantsummarystatistics. Dribble 0.32 0.35 0.20 0.13
Figure5(a)demonstratesthatshotpredictionoccursapprox- Cross 0.13 0.19 0.40 0.29
imately in line with actual data (see Figure 10 for comparison). Shot 0.07 0.12 0.27 0.54

<div class="figure">
    <img class="img-center" src="images/figure_10.png" alt="Figure 10" />
    <p class="img-caption"><strong>图10</strong>: ,anddefinedasfollows: A.3 PyTorchImplementationofModel</p>
</div>


Figure5(b)demonstratesthatmeanshotpredictionprobability
isgenerallyhigherclosertheoppositiongoal,givenashotispre-
4.3 Experiment3:AnalysisofPossession
dicted.Higherintensitiesinthelefthandsidehaveverylowsample
sizeandareinsignificant.Figure5(c)demonstratesmodeldiversity Utilisation
ofshotpredictionprobability,overgivenlocations.Diverseshot Attackmetricdevelopment.Ofthefouractionspredicted,shot
probabilitiesaremadeeveninareasofhighsamplesizeontheright andcrossmaybeconsideredtobe‘attacking’inintent.Acrossis
handside.SimilardiversitycanbeobservedinFigure9.Theseare definedasaballplayedfromtheoffensiveflanksaimedtowards
importantobservationsbecausetheydemonstratethatalthough ateammateintheareainfrontoftheopponent’sgoal,andashot
themodelisindeedpredictinghighershotprobabilitieswherewe isdefinedasanattempttowardstheopposition’sgoalwiththe
wouldexpectgiventheempiricalspatialdistribution,theirgenera- intentionofscoringagoal.2 Summingshotandcrosspredicted
tionisneithersolelyanindependentfunctionofspatialsourcenor probabilitiesthusgivesan‘attack’probability.Furthersumming
ofpredictedfeatures. per possession gives a measure of the weight of expectation of
ThemulticlassconfusionmatrixforthismodelisshowninTable attackaccumulatedduringeachpossession.
1.Forallactionsthemodel,onaverage,correctlyassignsthehighest In order to distinguish between attacking and non-attacking
probability,withtheexceptionofpasswhereitpredictsdribble possessions,thesummedattackprobabilitiesaremultipliedby−1
(𝑃 = 0.38)morethanpass(𝑃 = 0.36).Thiscouldpotentiallybe whennoattackevent(shotorcross)occurredinapossession.The
resolvedbyfinetuningtheweightsintheCELportionoftheloss reasonsfornotattackingvary,andthisapproachallowssomein-
function.Giventhelowactualclassoccurrenceofshot(2.2%),cross sightstobegained.Aweakerteammaysimplyhavenotmanagedto
(3.9%),anddribble(10.2%),theresultsarereasonable. createattackingopportunities(therebyincurringalowmagnitude
cumulativeexpectation,withnegativesign).Conversely,astronger
teammayhavebeenseekingtheoptimalopportunitytocommence
(a)ShotasPredictedNextAction
anattack(incurringahighmagnitudecumulativeexpectation,with
negativesign,intheprocess).Additionally,contextualfactorssuch
asgamestate(winning,losing,drawing)andperiodofthematch
mayalsoinfluenceateam’sattackingstrategy.
Foreaseofinterpretibility,percentilerankisappliedtoboth
positiveandnegativegroups,toprovideametricforeachpossession
withrange[−1,1],andtheresultantstatisticistermedposs-util.
(b)MeanP(Shot|ShotPredicted) Metricapplication.Themetricisappliedtoselectedteams
forthe2017/18seasonofLaLiga,Barcelona(leaguechampions),
Atlético Madrid (finished 2nd place), Real Madrid (3rd), Girona
(10th)andMálaga(20th).Distributionoverpossessionbyteamis
showninFigure6(a)andcanbeseentobelowerforpositivevalues
ofposs-util,sinceaminorityofpossessionsresultinanattack(23%).
Valuesofhighmagnitude,i.e.closeto-1or1,indicatepossessions
whereahighprobabilityofattackwasaccruedduringtheposses-
(c)StandardDeviationofP(Shot|ShotPredicted) sion,butwithonlythosewithpositivesignhavingresultedinan
attack(shotorcross).
BarcelonaandRealMadridarenoteworthyforhavingdistinct
distributionscomparedtotheotherteams.Theytendtogenerate
possessionswithhigherattackexpectation,asshownbythehigher
densityathighmagnitudepositiveandnegativevalues.Atlético
Madrid,Girona,andMálagahavesimilardistributionstoeachother.
AtléticoMadrid’ssimilaritytothesetwoteamsisperhapssurprising
giventhedifferencesinfinalleaguestandings,fromsecondposition
Figure 5: Spatial distribution of shot prediction statistics tomid-tableandlastplace.However,AtléticoMadridareknown

<div class="figure">
    <img class="img-center" src="images/figure_5.png" alt="Figure 5" />
    <p class="img-caption"><strong>图5</strong>: Spatial distribution of shot prediction statistics tomid-tableandlastplace.However,AtléticoMadridareknown</p>
</div>


(n=5,666). to play with an unusually defensive style for such a successful
2https://dataglossary.wyscout.com

Seq2Event:LearningtheLanguageofSoccerusingTransformer-basedMatchEventPrediction 
team.3 Theyconcededthefewestgoalspermatch(0.6)whichis (a)Distributionofposs-utiloverPossessions(n=23,951)
significantlybelowtheleagueaverage(1.6).Bycontrast,inattack,
theyscoredjustbelowtheaveragenumberofgoalspermatch(1.5
vs1.6),whichvalidatesourfindingthatAtléticoMadridaresimilar
tomoreaverageteamsinattack.Gironascored1.3goalspermatch
andfinishedmid-table.Málagascoredonly0.6goalspermatch,
despitehavingasimilardistributionofposs-utiltoAtléticoMadrid
andGirona,althoughaslastplacefinishersthismetrichighlights
theimportanceoftechnicalproficiencyinthefinalthirdandthe
abilityofplayerstoexecutespecificskills(shot,cross)toahigh
(b)DistributionofMean+’veposs-utiloverMatches(n=190)
level of accuracy and/or for players in these positions to make
optimaldecisionsonwhentoperformtheaction(s).Barcelonaand
RealMadridbothaccountedforthehighestnumberofgoalsscored
intheleague(2.61and2.47meangoalspermatchrespectively).
Usingposs-utiltopredictgoals.Goalsscoredwasselectedas
acomparisonmetric,asakeyandwelldefinedstatistic.Sum,mean,
andmedianofall,ofonlypositiveandofonlynegativeposs-util
wereanalysedforcorrelation(nineexperiments).Medianpositive
poss-utilhadthehighestPearsoncorrelation(𝑟 =0.47).Summed
Figure6:Analysisofbehaviourbyteamusingposs-util.
positiveposs-utilwasexpectedtobethemostintuitive,buthadonly
aweakcorrelation(𝑟 =0.17).Meanpositiveposs-utilshowedonly
aslightlylowercorrelation(𝑟 =0.46)thanmedian,andiseasierfor
asportsanalysttocomprehend,sowasthechosenmethod.Finally,
alineartransformofthemeanpositiveposs-utiloverpossessions
permatch(𝑝 ∈𝑀)againstactualgoalsovertheseasonwasused
toyieldpredictedgoals(𝑔ˆ),asshowninEquation2.
(cid:205) 𝑝
𝑔ˆ=6.5·
𝑝 𝑝 |𝑜 𝑝𝑠 |𝑠−𝑢𝑡𝑖𝑙
−1.5∀𝑝 ∈𝑀 :𝑝
𝑝𝑜𝑠𝑠−𝑢𝑡𝑖𝑙
>0 (2)
ValidationagainstactualgoalsandxG.Predictedgoalsby Figure7:DistributionoferrorofxGandpredictedgoalsby
poss-utilwasfoundtomoderatelycorrelatewithgoalsscored(𝑟 = meanposs-utilovermatches(n=190).
0.46overmatches),andfoundtostronglycorrelatewithxG4(𝑟
=
0.91).xGshowsmarginallystrongercorrelationwithactualgoals Table2:LaLiga2017/18MeanGoalsScoredperMatch
thanourmetric(𝑟 =0.57vs0.46).Figure7demonstratesfurther
similaritybetweenthetwometrics,althoughagainxGperforms
Team Actual poss-util poss-util xG
marginallybetter(RMSE1.29vs1.42).Table2showsaveryhigh
Goals Goals(Δ) Goals(Δ)
correlationofbothmetricsagainstactualswhenmeanaggregated
overtheseason,andwithourmetricperformingmarginallybetter
Barcelona 2.61 0.60 2.40(-0.21) 2.38(-0.23)
(poss-util 𝑟 = 0.98 vs xG 𝑟 = 0.97). Facets of team under and
AtléticoMadrid 1.53 0.46 1.49(-0.04) 1.32(-0.21)
RealMadrid 2.47 0.57 2.21(-0.26) 2.40(-0.07)
over-performanceagainstxGarevisibleinourmetricalso.Since
Girona 1.32 0.44 1.36(+0.04) 1.37(+0.05)
xG is a popular and relied upon model for the specific task of Málaga 0.63 0.40 1.10(+0.47) 0.94(+0.31)
goalscoredprediction[31],thesimilarityinpredictiveperformance
validatesourmetric.Byinduction,thisalsovalidatestheunderlying
Seq2Eventmodelfromwhichitisderived,andwhichwastrained
plotshowsBarcelonawinning3-1againstLeganeson7April2018.
onthemoregeneraltaskofnexteventprediction.
Overall,Barcelona’smeanpositiveposs-utilwas0.61,correspond-
ingto2.5predictedgoals(whichissimilartoxGmetric2.7).As

## 5 Modelapplicationtolaliga

depictedbytheorange10-minuterollingmeanlineinthenegative
Inthissection,weshowhowtheSeq2Eventmodelcanbeapplied region,Barcelonacanbeseentogeneratelotsofhighattacking
practicallyasateamprofilingmethodusingtheposs-utilmetric potentialpossessionsthatdonotconvertintoattack(linecloser
todeepengameunderstandingandderiveadditionalinsightsinto to-1)throughoutthematch.Whentheydoconverttoattack,the
attackingbehaviouroverthecourseofamatch. medianinthepositiveregioncanbeseentobehigh,althoughas
Matchtimelineview.Computingthemetricperpossession, denotedbytherollingmeanline,attacksoccurinfrequently.Look-
Figure8showstheevolutionofposs-utilovertwomatches.Thefirst ingatotherrelevantmatchstatistics,itisofnotethatBarcelona
tookandmaintainedaleadaftergoalsinthe26thand31stminutes;
3https://bleacherreport.com/articles/2589852-analysing-atletico-madrids-defensive-
structure-under-diego-simeone
andLeganesachievedanxGofonly0.65.Twomaininsightscanbe
4https://understat.com/league/la_liga/2017 gainedfromtheuseofthemodelinthisway:Barcelonahadahigh

 IanSimpson,RyanJ.Beal,DuncanLocke,andTimothyJ.Norman
(a)BarcelonaPossession(vsLeganes,07Apr2018,W3-1)
predictedgoalsandperformedapproximatelyaspredictedinthat
regard;theydidnotattackoften(denotedbytheblueline)despite
poss-util=0.98
accumulatingalotofhighpotentialpossessions.
ThesecondplotshowsRealMadridlosing0-1toVillarrealon13
January2018.Overall,RealMadrid’smeanpositiveposs-utilwas
0.49,correspondingto2.4predictedgoals(similartoxG metric
2.35).RealMadridcanbeseentoconsistentlygenerateamixtureof
possessionsofvaryingattackingpotentialandconversiontoattack
throughoutthematch.Villarrealscoredtheironlygoalinthe86th
minute.
Possessionoverview.Selectingpossessionsofinterestbasedon
themetric,Figure9(a)showsoneofBarcelona’shighestpossessions
Actualactions: pdpppppppppppppppppxxpppppx
byposs-util.Brightercolorsdenotehigherattackexpectation,and
Predictedactions: pdppxdxdpddddddppddxssxxxxs
thislongpossessioncanbeseentoaccumulateovermanyinstances (b)RealMadridPossession(vsVillarreal,13Jan2018,L0-1)
ofmoderateexpectationofattackattheedgeoftheopposition poss-util=0.39
third,followedbyseveralinstancesofhighexpectationofattack
intheoppositionleftcornerandpenaltyarea.Ultimately,across
wastakenwhichdidnotleadtoagoal,butapatientbuild-upof
possessionfollowedbyanattackrecordedahighmetricscore.By
contrast,thesecondplotshowsapossessionwithamoderatemetric
scoreof0.39.Adirectplay,initiallyexpectingalong-ballcrossfrom
thegoalkeeper,followedbyanunsuccessfulshot.
(a)Barcelonaposs-utiloverTime(vsLeganes,07Apr2018,W3-1)
Actualactions: ppppdps
Predictedactions: xpppdxx
Figure9:poss-utilmodelpredictedprobabilityofattackas
nextactionoveractual𝑥,𝑦andactualactionsvspredicted
mostlikelyactionovertwopossessions.Colorbyprobability
ofattack;‘S’and‘F’denotepossessionstartandfinish.For
(b)RealMadridposs-utiloverTime(vsVillarreal,13Jan2018,L0-1)
actiondecode,Table3refers.
1,401eventsrecordedonaveragepermatch,andeachactioncanbe
classifiedtoalearnttactic.Byanalysingtheoccurrenceoftacticsat
amacro-levelbothinaggregatebyteamandsequentially,insights
intotacticalutilisation,strategy,andimpactareachieveable.
GeneralPurposeMLModelsvsSpecificStatisticalModels.
Puttingeffortintotrainingmoreadvancedbutgeneralpurpose
Figure8:Evolutionofposs-utilovertime.Greypointsrepre-
probabilistic models is highlighted as a potential benefit to the
sentindividualpossessions;shadedcellsgiveavisualindi-
professionalsportsindustrywiththepotentialtoprovideteams
cationofpointdensityandmagnitude;10minutepositive
withversatilemodellingcapabilitieswhichcanadjustandadaptto
andnegativerollingmeanlinesshowninblueandorange.
derivecontemporarymetrics.Assportsevolve,thesemaybeused
Verticallinesindicategoalsfor(green)andagainst(red).
tohelpdeterminetheinfluenceoflawchangesorseasonstructures.
Themodelwetrainedwasgivenageneraltaskofpredicting
thenextmatcheventasparameterisedaccordingtothegeneralat-
6 DISCUSSION tackingfeaturesthatwereengineered.Fromthatbasis,aggregation
TeamBehaviourasDialect.Eventsinmostsportsgenerallyhap- andlinearregressionwerethenreadilyperformedonthemodel
pensequentially.Patternsappear,andstrategydecisionsaremade, probabilitiestogenerateausefulmetricwhichwehaveshowncor-
overmultipletemporalscales.Inthisway,sportcanbemodelled relateswithapopularspecificmetric.Fromtheotherprobabilities
andlearntusingRNNandTransformercomponentstoeffectively presentedbytheSeq2Event model,formulationofothermetrics
‘learnthelanguageofsports’.Inmuchthesamewaythatdialects shouldbereadilyachieveable,andbymodifyinginparticularthe
canbedetectedbyanalysingdifferencesinvocabularyfromthat finallayersofthemodel,andtheeventstreaming,thegeneralprin-
expected,differenttacticscanbedetectedbyanalysingdifferences ciplecouldbeputtouseontasksotherthanexaminingattacking
inactionsfromthatexpected.Inoursoccerdataset,therewere styles.

Seq2Event:LearningtheLanguageofSoccerusingTransformer-basedMatchEventPrediction 
6.1 FutureWork [6] DanielBerrar,PhilippeLopes,JesseDavis,andWernerDubitzky.2019.Guest
FurtherLeveragingModelOutputs.Initialworkwasconducted
(e 2d 0i 1to 9r )i ,a 1l –:s 7p .ecialissueonmachinelearningforsoccer.MachineLearning108,1
toutilisethespatialfeaturesthatarepredicted.Analysisofpredic- [7] RobertoCahuantzi,XinyeChen,andStefanGüttel.2021.AcomparisonofLSTM
tionerrorin𝑥 locationshowednosignificantdifferenceinmeans andGRUnetworksforlearningsymbolicsequences.(2021).
[8] JunyoungChung,CaglarGulcehre,KyunghyunCho,andYoshuaBengio.2014.
across the five teams analysed in Section 5. However, the stan-
Empiricalevaluationofgatedrecurrentneuralnetworksonsequencemodeling.
darddeviationoferrorcorrelatedweaklywithstandingsandgoals InNIPS2014WorkshoponDeepLearning,December2014.
scored(standarddeviationsof0.157,0.176,0.164,0.190,0.193for [9] TomDecroos.2020.SoccerAnalyticsMeetsArtificialIntelligence:LearningValue
andStylefromSoccerEventStreamData.Ph.D.Dissertation.KULeuven.
BarcelonatoMálagain17/18LaLigafinalstandingrespectively). [10] TomDecroos,LotteBransen,JanVanHaaren,andJesseDavis.2019. Actions
Essentially,themodelwasfoundtobeabletopredictthenext𝑥 speaklouderthangoals:Valuingplayeractionsinsoccer.InProceedingsofthe25th
position of Barcelona and RealMadrid more preciselythan the
ACMSIG IanSimpson,RyanJ.Beal,DuncanLocke,andTimothyJ.Norman
A REPRODUCIBILITY
A.1 DataPreparation
UseoftheWyScoutOpenAccessDataset.Thecompetitions,
matches,teamsandeventsJSONfileswereusedforthisresearch.
Sincethisresearchwasfocusedonteamperformance,playerfiles
werenotused,butitisnotedthatthisinformationisreadilyavail-
abletobeincorporatedintomodelsforfutureresearchonplayer
behaviour.
Theeventsfilerecordsmatchevents,andareclassifiedaccording
to21‘event’categoriesand78‘subevent’categories,withseveral
hundredpossibletagsprovidingfurtherdetail.Tagtypesvarysig-
nificantlyintheirscopeofapplicationacrossthedataset,inthat
somearespecificandonlyusedtoelaborateoneevent/subevent
type,whilstothersareinterpretablemoregenerically.
ActionFeatureEngineering.Asimplifiedencodingofmatch
event attributes was necessary in order to define an associated
categoricalactionlabelofacceptabledimensionality.Iftraining
on a far larger set, such reduction might not be necessary; the
event,subevent,andtaginformationcouldbepasseddirectlyto
theembeddinglayerfromwhichtherelevanceofthisinformation
couldbelearntbythemodel.However,fortherelativelycompactset
usedinthisresearch,areductionindimensionalitywasconsidered
Figure 10: Spatial distribution of occurrence for all engi-

<div class="figure">
    <img class="img-center" src="images/figure_10.png" alt="Figure 10" />
    <p class="img-caption"><strong>图10</strong>: ,anddefinedasfollows: A.3 PyTorchImplementationofModel</p>
</div>


necessary.
neeredsourcefeatures(n=96,850).Foractionfeatures,density
Qualitativeanalysiswasfirstconductedtofilterandgroupmatch
ofoccurrenceisshown.Forcontinuousfeatures,meanvalue
eventsthatappropriatelycharaceteriseattackingplay.Makingref-
isshown.Attackingdirectionisfromlefttoright.
erencetotheencodingsperformedby[9][27]andreflectingon
theobjectiveofmodellingattackingstyles,aninitialencodingwas
madeasshowninTable3.Thisencodingprovidedarichsequential DefiningModellingSetsSamplingofallsevenavailablecom-
diversity,andgaveanintuitivepicturetothehumananalystwhen petitionsonhigh/moderate/lowplacingteamswasperformedto
observingdataencodedinthisway.However,thecategorysupport capturearepresentativesampleofsuccess,andisshowninTable6.
wastooimbalancedortooweakinmanycases,e.g.passaccounts A50/6/44train/validation/testratiowasusedwithteamsexclusive
forasmuchas49.3%ofeventsandpenaltykickaccountsforonly totrain/validationandtestsets.Ahightestweightingwasusedto
0.02%ofevents.Thus,acoarserencodingwasdeemednecessary. ensurearepresentativecross-sectionofteams,seasonstatesand
Forthe‘final’projectencoding,eventsweregroupedintofour leagues.
categories:pass(‘p’),dribble(‘d’),cross(‘x’),andshot(‘s’).Three
additionaleventtypeswereaddedforpossessionandmatchcon- A.2 HyperparameterSearch
text:goalscored(‘g’),possessionend(‘_’),andmatchend(’@‘). Table4outlinestheschemeusedforhyperparametersearch.An
initialwidemanualsearchwasconducted,afterwhichafocused
ContinuousFeatureEngineering.Eventlocationco-ordinates combinatorialgridsearchwasconductedonthehyperparameters
andmatchtimeareprovidedinthesourcedata.Fromthese,ad- showninbold.Table5showsthehyperparametersofthetop10
ditionalfeatureswereengineeredtohelptrainingbyproviding Seq2Event models.Thethirdbestmodel,aTransformervariant,
informationonknownsourcesofvariance[9],andbyproviding whichhadbeentrainedonthedatashowninTable6wasthen
contextacrossdifferenttemporalscales(e.g.𝑥,𝑡ℎ𝑒𝑡𝑎𝑔,and𝑠𝑐𝑟𝑎𝑑 usedtopredicttheeventsforthefiveLaLigateamsoverthewhole
changeovertimeatdifferentordersofmagnitude).Thisresultedin season,forwhichresultsarepresentedinthemainpaper.
tennormalisedfeatures,withspatialdistributionshowninFigure
10,anddefinedasfollows: A.3 PyTorchImplementationofModel
• 𝑥,𝑦 ∈ [0,1]:eventlocationco-ordinates. ModellingwasperformedusingPyTorch,andcodeformodelrepro-
• 𝑑𝑒𝑙𝑡𝑎𝑥,𝑑𝑒𝑙𝑡𝑎𝑦 ∈ [−1,1]:differencein𝑥,𝑦sinceprevious. ductionissharedbyus[28].Twonotebooksarepresented:adata
√
• 𝑠 ∈ [0, 2]:distancesincepreviousevent. preparationandfeatureengineeringnotebook,andamodelling
√
• 𝑠𝑔∈ [0, 1.5]:distancetocentreofoppositiongoal. notebook.
• 𝑡ℎ𝑒𝑡𝑎𝑔∈ [𝜋/2,𝜋]:anglefromcentreofoppositiongoal.
• 𝑇 ∈ [0,1]:eventmatchtime.
• 𝑑𝑒𝑙𝑡𝑎𝑇:time(normalised)sincepreviousevent.
• 𝑠𝑐𝑟𝑎𝑑 ∈ [−6,6]:currentscoreadvantage:numberofgoals
ahead(positive)orbehind(negative).

Seq2Event:LearningtheLanguageofSoccerusingTransformer-basedMatchEventPrediction 
Table6:SchemeforModellingSets Table3:WyScoutEventtoProjectEncodingMapping
Competition/Team/Position Trg Val Tst WyScoutEvent/SubEventDescription Project Project Proportion of
Encod- Encod- allevents
FIFAWC2018 12 1 11 ing ing
France(C) 3 1 - (initial) (final)
Croatia(F) - - 2 Pass(Handpass) p 0.43%
Colombia(R16) 3 - - Pass(Headpass) p 2.98%
Denmark(R16) - - 3 Pass(Highpass) p 4.01%
SouthKorea(G3) 3 - - Pass(Launch) p 1.41%
Tunisia(G3) - - 3 Pass(Simplepass) p p 39.55%
Panama(G4) 3 - - Pass(Smartpass) p (Pass) 0.93% 56.1%
Poland(G4) - - 3 Othersontheball(Clearance) o 1.75%
FreeKick(Goalkick) 0 0.98%
UEFAEuro2016 12 1 11
FreeKick(Throwin) 1 2.62%
Portugal(F) 3 - - FreeKick(FreeKick) 3 1.40%
Wales(SF) - - 3
Hungary(R16) 3 1 - Duel(Groundattackingduel) d d 8.61%
Switzerland(R16) - - 2 Othersontheball(Acceleration) t (Dribble) 0.80% 14.8%
RepublicofIreland(R16) 3 - - Othersontheball(Touch) t 5.37%
Albania(G3) - - 3 Pass(Cross) x x 1.92%
CzechRepublic(G4) 3 - - FreeKick(Corner) 2 (Cross) 0.59% 2.8%
Sweden(G4) - - 3 FreeKick(Freekickcross) 4 0.27%
EnglishPL2017/18 9 1 8 Shot(Shot) s s 1.32%
ManchesterCity(1) 3 - - FreeKick(Freekickshot) 5 (Shot) 0.07% 1.4%
ManchesterUnited(2) - - 3 FreeKick(Penalty) 6 0.02%
NewcastleUnited(10) - - 3 Foul(Foul) f 1.45%
CrystalPalace(11) 3 1 - Foul(Handfoul) f 0.06%
StokeCity(19) - - 2 Foul(Latecardfoul) f 0.01%
WestBromwichAlbion(20) 3 - - Foul(Outofgamefoul) f 0.02%
Foul(Protest) f 0.02%
FrenchLigue12017/18 9 1 8
Foul(Simulation) f 0.00%
ParisSaint-Germain(1) 3 1 - Foul(Timelostfoul) f 0.01%
Monaco(2) - - 2 Foul(ViolentFoul) f n/a 0.00%
Montpellier(10) - - 3 Offside(noSubEvents) f (Foulor 0.25% 25.0%
Dijon(11) 3 - - Duel(Airduel) n/a Defensive 5.18%
Troyes(19) - - 3 Duel(Grounddefendingduel) n/a Actions) 8.57%
Metz(20) 3 - - Duel(Groundlooseballduel) n/a 4.67%
Gkprleavingline(Gkprleavingline) n/a 0.19%
GermanBundesliga2017/18 9 1 8 Interruption(Balloutofthefield) n/a 3.97%
BayernMunich(1) 3 - - Interruption(Whistle) n/a 0.03%
Schalke04(2) - - 3 Saveattempt(Reflexes) n/a 0.33%
BorussiaMönchengladbach(9) - - 2 Saveattempt(Saveattempt) n/a 0.21%
HerthaBSC(10) 3 1 -
HamburgerSV(17) - - 3
Table4:SchemeforHyperparameterSearch
1.FCKöln(18) 3 - -
SpanishLaLiga2017/18 9 1 8
Barcelona(1) 3 - - Hyperparameter Values(bold=focusedsearch)
AtléticoMadrid(2) - - 3 ModelType Elman-RNN,LSTM,GRU,Transformer
Girona(10) - - 3 SequenceLength 5,10,40,100
Espanyol(11) 3 - - Steplength 1,20
LasPalmas(19) - - 2 ActionEmbed.Dim. 1,5,7,20
Málaga(20) 3 1 - Cont.Feat.Embed.Dim. 1,3,5,9,10,20
ItalianSerieA2017/18 9 1 8
Numberoflayers 1,2,5
Hidden/FeedforwardDim. 8,16,256,1024,2048,4096,8192,16384,32768
Juventus(1) 3 1 - RNNDirectionality Uni-Directional,Bi-Directional
Napoli(2) - - 2 TransformerNum.Heads 1,2,17
Sampdoria(10) - - 3
Sassuolo(11) 3 - -
HellasVerona(19) - - 3 Table5:ModelsbyTestLoss:Top10plusSelectedBaselines
Benevento(20) 3 - -
Total 69 7 62
Variant Seq. Step Act. Cont.Num.Hdn. Dir. Loss
Len. Emb.Emb.Lyrs.Dim. /Num (Rank)
Dim. Dim. Heads
LSTM 100 1 7 10 1 8 UNIDIR 0.332(1)
LSTM 100 1 7 10 1 8 BIDIR 0.344(2)
Transformer 40 1 7 10 1 8 1 0.362(3)
Transformer 40 1 7 10 1 16 1 0.364(4)
Transformer 10 20 3 5 1 8192 1 0.368(5)
Transformer 10 20 1 20 1 8192 1 0.37(6)
Transformer 10 20 1 5 1 8192 1 0.372(7)
Transformer 10 20 5 3 1 8192 1 0.373(8)
Transformer 10 20 5 5 1 8192 1 0.373(9)
Transformer 10 20 3 3 1 8192 1 0.373(10)
Baseline:AR+MCOrder1 1 1 7 2 - - - 0.667(146)
Baseline:AR+MCOrder5 5 1 7 2 - - - 0.679(152)
Baseline:Lag1(t=t-1) 1 1 7 2 - - - 0.708(156)

## 参考文献

<a id="ref-29"></a>
**[29]** KarunSingh.2021.IntroducingExpectedThreat(xT). RetrievedDecember27, 2021fromhttps://karun.in/blog/expected-threat.html

<a id="ref-1"></a>
**[1]** DavidAdams,RylandMorgans,JoaoSacramento,StuartMorgan,andMorganD [30] DaniloSorano,FabioCarrara,PaoloCintia,FabrizioFalchi,andLucaPappalardo. Williams.2013.Successfulshortpassingfrequencyofdefendersdifferentiates 2020.AutomaticPassAnnotationfromSoccerVideoStreamsBasedonObject betweentopandbottomfourEnglishPremierLeagueteams.InternationalJournal DetectionandLSTM.InJointEuropeanConferenceonMachineLearningand ofPerformanceAnalysisinSport13,3(2013),653–668. KnowledgeDiscoveryinDatabases.Springer,475–490.

<a id="ref-2"></a>
**[2]** RocksonAgyeman,RafiqMuhammad,andGyuSangChoi.2019.Soccervideo [31] WilliamSpearman.2018.Beyondexpectedgoals.InProceedingsofthe12thMIT summarizationusingdeeplearning.In2019IEEEConferenceonMultimedia SloanSportsAnalyticsConference. InformationProcessingandRetrieval(MIPR).IEEE,270–273. [32] AshishVaswani,NoamShazeer,NikiParmar,JakobUszkoreit,LlionJones,

<a id="ref-3"></a>
**[3]** RyanBeal,GeorgiosChalkiadakis,TimothyJNorman,andSarvapaliDRamchurn. AidanNGomez,ŁukaszKaiser,andIlliaPolosukhin.2017. Attentionisall 2020.OptimisingGameTacticsforFootball.InProceedingsofthe19thInternational youneed.InAdvancesinneuralinformationprocessingsystems.5998–6008. ConferenceonAutonomousAgentsandMultiAgentSystems.141–149. [33] ZhiweiWang,YaoMa,ZitaoLiu,andJiliangTang.2019.R-transformer:Recurrent

<a id="ref-4"></a>
**[4]** RyanBeal,GeorgiosChalkiadakis,TimothyJNorman,andSarvapaliDRamchurn. neuralnetworkenhancedtransformer.(2019). 2021.OptimisingLong-TermOutcomesusingReal-WorldFluentObjectives:An [34] NeilWatson,ShariefHendricks,TheodorStewart,andIanDurbach.2021.Inte- ApplicationtoFootball.InProceedingsofthe20thInternationalConferenceon gratingmachinelearninganddecisionsupportintacticaldecision-makingin AutonomousAgentsandMultiAgentSystems.196–204. rugbyunion.JournaloftheOperationalResearchSociety72,10(2021),2274–2285.

<a id="ref-5"></a>
**[5]** RyanBeal,TimothyJNorman,andSarvapaliDRamchurn.2019. Artificial [35] QiyunZhang,XuyunZhang,HongshengHu,CaizhongLi,YinpingLin,and intelligenceforteamsports:asurvey. TheKnowledgeEngineeringReview34 RuiMa.2021. Sportsmatchpredictionmodelfortrainingandexerciseusing (2019). attention-basedLSTMnetwork.DigitalCommunicationsandNetworks(2021). KDD’22,August14–18,2022,Washington,DC,USA. IanSimpson,RyanJ.Beal,DuncanLocke,andTimothyJ.Norman A REPRODUCIBILITY A.1 DataPreparation UseoftheWyScoutOpenAccessDataset.Thecompetitions, matches,teamsandeventsJSONfileswereusedforthisresearch. Sincethisresearchwasfocusedonteamperformance,playerfiles werenotused,butitisnotedthatthisinformationisreadilyavail- abletobeincorporatedintomodelsforfutureresearchonplayer behaviour. Theeventsfilerecordsmatchevents,andareclassifiedaccording to21‘event’categoriesand78‘subevent’categories,withseveral hundredpossibletagsprovidingfurtherdetail.Tagtypesvarysig- nificantlyintheirscopeofapplicationacrossthedataset,inthat somearespecificandonlyusedtoelaborateoneevent/subevent type,whilstothersareinterpretablemoregenerically. ActionFeatureEngineering.Asimplifiedencodingofmatch event attributes was necessary in order to define an associated categoricalactionlabelofacceptabledimensionality.Iftraining on a far larger set, such reduction might not be necessary; the event,subevent,andtaginformationcouldbepasseddirectlyto theembeddinglayerfromwhichtherelevanceofthisinformation couldbelearntbythemodel.However,fortherelativelycompactset usedinthisresearch,areductionindimensionalitywasconsidered Figure 10: Spatial distribution of occurrence for all engi- necessary. neeredsourcefeatures(n=96,850).Foractionfeatures,density Qualitativeanalysiswasfirstconductedtofilterandgroupmatch ofoccurrenceisshown.Forcontinuousfeatures,meanvalue eventsthatappropriatelycharaceteriseattackingplay.Makingref- isshown.Attackingdirectionisfromlefttoright. erencetotheencodingsperformedby[9][27]andreflectingon theobjectiveofmodellingattackingstyles,aninitialencodingwas madeasshowninTable3.Thisencodingprovidedarichsequential DefiningModellingSetsSamplingofallsevenavailablecom- diversity,andgaveanintuitivepicturetothehumananalystwhen petitionsonhigh/moderate/lowplacingteamswasperformedto observingdataencodedinthisway.However,thecategorysupport capturearepresentativesampleofsuccess,andisshowninTable6. wastooimbalancedortooweakinmanycases,e.g.passaccounts A50/6/44train/validation/testratiowasusedwithteamsexclusive forasmuchas49.3%ofeventsandpenaltykickaccountsforonly totrain/validationandtestsets.Ahightestweightingwasusedto 0.02%ofevents.Thus,acoarserencodingwasdeemednecessary. ensurearepresentativecross-sectionofteams,seasonstatesand Forthe‘final’projectencoding,eventsweregroupedintofour leagues. categories:pass(‘p’),dribble(‘d’),cross(‘x’),andshot(‘s’).Three additionaleventtypeswereaddedforpossessionandmatchcon- A.2 HyperparameterSearch text:goalscored(‘g’),possessionend(‘_’),andmatchend(’@‘). Table4outlinestheschemeusedforhyperparametersearch.An initialwidemanualsearchwasconducted,afterwhichafocused ContinuousFeatureEngineering.Eventlocationco-ordinates combinatorialgridsearchwasconductedonthehyperparameters andmatchtimeareprovidedinthesourcedata.Fromthese,ad- showninbold.Table5showsthehyperparametersofthetop10 ditionalfeatureswereengineeredtohelptrainingbyproviding Seq2Event models.Thethirdbestmodel,aTransformervariant, informationonknownsourcesofvariance[9],andbyproviding whichhadbeentrainedonthedatashowninTable6wasthen contextacrossdifferenttemporalscales(e.g.𝑥,𝑡ℎ𝑒𝑡𝑎𝑔,and𝑠𝑐𝑟𝑎𝑑 usedtopredicttheeventsforthefiveLaLigateamsoverthewhole changeovertimeatdifferentordersofmagnitude).Thisresultedin season,forwhichresultsarepresentedinthemainpaper. tennormalisedfeatures,withspatialdistributionshowninFigure 10,anddefinedasfollows: A.3 PyTorchImplementationofModel • 𝑥,𝑦 ∈ [0,1]:eventlocationco-ordinates. ModellingwasperformedusingPyTorch,andcodeformodelrepro- • 𝑑𝑒𝑙𝑡𝑎𝑥,𝑑𝑒𝑙𝑡𝑎𝑦 ∈ [−1,1]:differencein𝑥,𝑦sinceprevious. ductionissharedbyus[28].Twonotebooksarepresented:adata √ • 𝑠 ∈ [0, 2]:distancesincepreviousevent. preparationandfeatureengineeringnotebook,andamodelling √ • 𝑠𝑔∈ [0, 1.5]:distancetocentreofoppositiongoal. notebook. • 𝑡ℎ𝑒𝑡𝑎𝑔∈ [𝜋/2,𝜋]:anglefromcentreofoppositiongoal. • 𝑇 ∈ [0,1]:eventmatchtime. • 𝑑𝑒𝑙𝑡𝑎𝑇:time(normalised)sincepreviousevent. • 𝑠𝑐𝑟𝑎𝑑 ∈ [−6,6]:currentscoreadvantage:numberofgoals ahead(positive)orbehind(negative). Seq2Event:LearningtheLanguageofSoccerusingTransformer-basedMatchEventPrediction KDD’22,August14–18,2022,Washington,DC,USA. Table6:SchemeforModellingSets Table3:WyScoutEventtoProjectEncodingMapping Competition/Team/Position Trg Val Tst WyScoutEvent/SubEventDescription Project Project Proportion of Encod- Encod- allevents FIFAWC2018 12 1 11 ing ing France(C) 3 1 - (initial) (final) Croatia(F) - - 2 Pass(Handpass) p 0.43% Colombia(R16) 3 - - Pass(Headpass) p 2.98% Denmark(R16) - - 3 Pass(Highpass) p 4.01% SouthKorea(G3) 3 - - Pass(Launch) p 1.41% Tunisia(G3) - - 3 Pass(Simplepass) p p 39.55% Panama(G4) 3 - - Pass(Smartpass) p (Pass) 0.93% 56.1% Poland(G4) - - 3 Othersontheball(Clearance) o 1.75% FreeKick(Goalkick) 0 0.98% UEFAEuro2016 12 1 11 FreeKick(Throwin) 1 2.62% Portugal(F) 3 - - FreeKick(FreeKick) 3 1.40% Wales(SF) - - 3 Hungary(R16) 3 1 - Duel(Groundattackingduel) d d 8.61% Switzerland(R16) - - 2 Othersontheball(Acceleration) t (Dribble) 0.80% 14.8% RepublicofIreland(R16) 3 - - Othersontheball(Touch) t 5.37% Albania(G3) - - 3 Pass(Cross) x x 1.92% CzechRepublic(G4) 3 - - FreeKick(Corner) 2 (Cross) 0.59% 2.8% Sweden(G4) - - 3 FreeKick(Freekickcross) 4 0.27% EnglishPL2017/18 9 1 8 Shot(Shot) s s 1.32% ManchesterCity(1) 3 - - FreeKick(Freekickshot) 5 (Shot) 0.07% 1.4% ManchesterUnited(2) - - 3 FreeKick(Penalty) 6 0.02% NewcastleUnited(10) - - 3 Foul(Foul) f 1.45% CrystalPalace(11) 3 1 - Foul(Handfoul) f 0.06% StokeCity(19) - - 2 Foul(Latecardfoul) f 0.01% WestBromwichAlbion(20) 3 - - Foul(Outofgamefoul) f 0.02% Foul(Protest) f 0.02% FrenchLigue12017/18 9 1 8 Foul(Simulation) f 0.00% ParisSaint-Germain(1) 3 1 - Foul(Timelostfoul) f 0.01% Monaco(2) - - 2 Foul(ViolentFoul) f n/a 0.00% Montpellier(10) - - 3 Offside(noSubEvents) f (Foulor 0.25% 25.0% Dijon(11) 3 - - Duel(Airduel) n/a Defensive 5.18% Troyes(19) - - 3 Duel(Grounddefendingduel) n/a Actions) 8.57% Metz(20) 3 - - Duel(Groundlooseballduel) n/a 4.67% Gkprleavingline(Gkprleavingline) n/a 0.19% GermanBundesliga2017/18 9 1 8 Interruption(Balloutofthefield) n/a 3.97% BayernMunich(1) 3 - - Interruption(Whistle) n/a 0.03% Schalke04(2) - - 3 Saveattempt(Reflexes) n/a 0.33% BorussiaMönchengladbach(9) - - 2 Saveattempt(Saveattempt) n/a 0.21% HerthaBSC(10) 3 1 - HamburgerSV(17) - - 3 Table4:SchemeforHyperparameterSearch 1.FCKöln(18) 3 - - SpanishLaLiga2017/18 9 1 8 Barcelona(1) 3 - - Hyperparameter Values(bold=focusedsearch) AtléticoMadrid(2) - - 3 ModelType Elman-RNN,LSTM,GRU,Transformer Girona(10) - - 3 SequenceLength 5,10,40,100 Espanyol(11) 3 - - Steplength 1,20 LasPalmas(19) - - 2 ActionEmbed.Dim. 1,5,7,20 Málaga(20) 3 1 - Cont.Feat.Embed.Dim. 1,3,5,9,10,20 ItalianSerieA2017/18 9 1 8 Numberoflayers 1,2,5 Hidden/FeedforwardDim. 8,16,256,1024,2048,4096,8192,16384,32768 Juventus(1) 3 1 - RNNDirectionality Uni-Directional,Bi-Directional Napoli(2) - - 2 TransformerNum.Heads 1,2,17 Sampdoria(10) - - 3 Sassuolo(11) 3 - - HellasVerona(19) - - 3 Table5:ModelsbyTestLoss:Top10plusSelectedBaselines Benevento(20) 3 - - Total 69 7 62 Variant Seq. Step Act. Cont.Num.Hdn. Dir. Loss Len. Emb.Emb.Lyrs.Dim. /Num (Rank) Dim. Dim. Heads LSTM 100 1 7 10 1 8 UNIDIR 0.332(1) LSTM 100 1 7 10 1 8 BIDIR 0.344(2) Transformer 40 1 7 10 1 8 1 0.362(3) Transformer 40 1 7 10 1 16 1 0.364(4) Transformer 10 20 3 5 1 8192 1 0.368(5) Transformer 10 20 1 20 1 8192 1 0.37(6) Transformer 10 20 1 5 1 8192 1 0.372(7) Transformer 10 20 5 3 1 8192 1 0.373(8) Transformer 10 20 5 5 1 8192 1 0.373(9) Transformer 10 20 3 3 1 8192 1 0.373(10) Baseline:AR+MCOrder1 1 1 7 2 - - - 0.667(146) Baseline:AR+MCOrder5 5 1 7 2 - - - 0.679(152) Baseline:Lag1(t=t-1) 1 1 7 2 - - - 0.708(156)


---

> 本文档由 PDF 自动转换生成 | 转换时间: 2025-11-09 02:05:36
