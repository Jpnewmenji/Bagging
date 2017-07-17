# Bagging
def Bagging():
    
                   
    def splitindex(fetu):   
            Gini_Value=np.empty([1,len(top500)], dtype=int)*0.0
            for i in range(0,len(top500)):#
              if len(fetu[fetu[:,i]==1,-1])==0 or len(fetu[fetu[:,i]==0,-1])==0:    
   
                    Gini_Value[0,i]=1
              else:        
                    Posit_Posit=np.sum(fetu[fetu[:,i]==1,len(top500)])/len(fetu[fetu[:,i]==1,len(top500)])
                    
                    #print (Posit_Posit)
                   
                    Posit_Negat=1-Posit_Posit
                    Negat_Posit=np.sum(fetu[fetu[:,i]==0,len(top500)])/len(fetu[fetu[:,i]==0,len(top500)])
                    #print (Negat_Posit)
                    Negat_Negat=1-Negat_Posit
                     
                    Gini_Value[0,i]=((1-(Posit_Posit**2)-(Posit_Negat**2))*(len(fetu[fetu[:,i]==1,len(top500)])/len(fetu)))+((1-(Negat_Posit**2)-(Negat_Negat**2))*(len(fetu[fetu[:,i]==0,len(top500)])/len(fetu)))
                #print (Gini) 
            return(np.argmin(Gini_Value))
    
           
    
    def get_split(feturematrix):
        
        split=splitindex(feturematrix)
        group =list(feturematrix[feturematrix[:,split]==1,]),list(feturematrix[feturematrix[:,split]==0,])
        return {'index':split,  'groups':group}
    
    def to_terminal(group):
    	outcomes = [row[-1] for row in group]
    	return max(set(outcomes), key=outcomes.count)
 

    def split(node, max_depth, min_size, depth):
    	left, right = node['groups']
    	del(node['groups'])
    	
    	if not left or not right:
    		node['left'] = node['right'] = to_terminal(left + right)
    		return
    	
    	if depth >= max_depth:
    		node['left'], node['right'] = to_terminal(left), to_terminal(right)
    		return
    	
    	if len(left) <= min_size:
    		node['left'] = to_terminal(left)
    	else:
    		node['left'] = get_split(np.asarray(left))
    		split(node['left'], max_depth, min_size, depth+1)
    
    	if len(right) <= min_size:
    		node['right'] = to_terminal(right)
    	else:
    		node['right'] = get_split(np.asarray(right))
    		split(node['right'], max_depth, min_size, depth+1)
     
    def build_tree(train, max_depth, min_size):
    	root = get_split(train)
    	split(root, max_depth, min_size, 0)
    	return root    

        
    def predict(node, row):
    	if row[node['index']]==1:
    		if isinstance(node['left'], dict):
    			return predict(node['left'], row)
    		else:
    			return node['left']
    	else:
    		if isinstance(node['right'], dict):
    			return predict(node['right'], row)
    		else:
    			return node['right']
 
 
    def subsample(dataset, ratio):
    	sample = list()
    	n_sample = round(len(dataset) * ratio)
    	while len(sample) < n_sample:
    		index = randrange(len(dataset))
    		sample.append(dataset[index])
    	return sample
        
     
    def bagging_predict(trees, row):
    	predictions = [predict(tree, row) for tree in trees]
    	return max(set(predictions), key=predictions.count)
    
    
    def bagging(train,test,max_depth,min_size,sample_size, n_trees):
        trees=list()
        for i in range(n_trees):
            sample=subsample(list(feature),sample_size)
            tree = build_tree(np.asarray(sample), max_depth, min_size)
            trees.append(tree)
            #print(i)
        predictions=[bagging_predict(trees,row) for row in list (test_matrix)]
        count=0
        for i in range(0,len(predictions)):
            
         if predictions[i]==Test_array[i,1]:
           count=count+1
        percent=count / len(predictions) 
        Zeroloss=1-percent
        print(percent)
        return (Zeroloss)    
    
    array=np.empty([len(Train), 1], dtype=int)*0
    for i in range (0,len(Train)):
       array[i,0]=Train_array[i,1]
    feature=np.concatenate((matrix,array),axis=1)
    Zeroerror=bagging(feature,test_matrix,10, 10, 1,50) 
    print ("ZERO_ONE_LOSS BAGGING %s"%Zeroerror) 
    return(Zeroerror) 

