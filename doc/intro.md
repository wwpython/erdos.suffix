# Introduction to erdos.suffixtree

TODO: write [great documentation](http://jacobian.org/writing/what-to-write/)




(ns erdos.suffixtree)

;; sources https://github.com/kvh/Python-Suffix-Tree
;; generalize using the ideas: http://stackoverflow.com/questions/28278802/ukkonens-algorithm-for-generalized-suffix-trees

(defn- ->node []
  {:suffix_node -1})

;; first char idx, last char idx, source node idx, dest node idx
(defn- ->edge [[fci lci] [sni dni]]
  {:fci fci, :lci lci, :sni sni, :dni dni})

(defn- edge-length [n]
  (- (:lci n) (:fci n)))

;; source node indes, first char index, last char index
(defn- ->suffix [sni, fci, lci]
  {:sni sni, :fci fci, :lci lci})

(def suffix-length edge-length)

(defn- explicit? [suffix]
  (> (:fci suffix) (:lci suffix)))

(def implicit (complement explicit?))

(defn- insert-edge [self, edge]
  (assoc-in self [:edges [(:sni edge) (nth (:str self) (:fci edge))]] edge))

(defn- remove-edge [self {:keys [sni fci]}]
  (assoc-in self [:edges [sni (nth (:str self) fci)]] nil))

(defn- split-edge [self, edge, suffix]
  (let [self (update-in self [:nodes] conj (->node))
        e (->edge [(:fci edge)    (+ (:fci edge) (suffix-length suffix) )]
                  [(:sni suffix)  (dec (count (:nodes self)))])
        self (remove-edge self edge)
        self (insert-edge self e)
        self (assoc-in self [:nodes (:dni e) :suffix_node] (:sni suffix))
        edge (update-in edge [:fci] + (inc (suffix-length suffix)))
        edge (assoc-in edge [:sni] (:dni e))
        self (insert-edge self edge)]
    [self, (:dni e)]))

(defn- canonize-suffix- [self, suffix]
  (if-not (explicit? suffix)
    (let [e (get-in self [:edges [(:sni suffix) (nth (:str self) (:fci suffix))]])]
      (if (<= (edge-length e) (suffix-length suffix))
        (recur self
               (assoc suffix
                 :fci (+ (:fci suffix) 1 (edge-length e))
                 :sni (:dni e)))
        suffix))
    suffix))

;; kenyelmi funkcio, ugyis csak innen hivjuk.
(defn- canonize-suffix [self]
  (assoc self :active (canonize-suffix- self (:active self))))


(defn- add-prefix [self, lci]
  (let [self (atom self)
        last_parent_node (atom -1)
        parent_node (atom 0)]
    (try
      (while true
        (do
          (reset! parent_node (-> @self :active :sni))
          (if (explicit? (:active @self))
            (if (contains? (:edges @self) [(-> @self :active :sni) (nth (:str @self) lci)])
              (throw (InterruptedException.)))
            (let [e (get (:edges @self) [(-> @self :active :sni) (nth (:str @self) (-> @self :active :fci))])]
              (if (= (nth (:str @self) (+ (:fci e) (-> @self :active edge-length) 1))
                     (-> (:str @self) (nth lci)))
                (throw (InterruptedException.)))
              (let [[s idx] (split-edge @self, e (:active @self))]
                (reset! self s)
                (reset! parent_node idx))))

          (as-> @self *
                (update-in * [:nodes] conj  (->node))
                (insert-edge * (->edge [lci, (:n *)] [@parent_node (dec (count (:nodes *)))]))
                (cond-> *
                        (pos? @last_parent_node)
                        (assoc-in [:nodes @last_parent_node :suffix_node] @parent_node))

                (if (zero? (-> * :active :sni (or 0)))
                  (update-in * [:active :fci] inc)
                  (assoc-in * [:active :sni]
                         (get-in * [:nodes (-> * :active :sni) :suffix_node])))
                (canonize-suffix *)
                (reset! self *))

          (reset! last_parent_node @parent_node)))

      (catch InterruptedException e nil)) ;; end of while loop

    (-> @self
        (cond-> (pos? (or @last_parent_node -1))
                (assoc-in [:nodes @last_parent_node :suffix_node] @parent_node))
        (update-in [:active :lci] inc)
        (canonize-suffix))))

(defn ->tree [s]
  (let [t {:str s
           :n (dec (count s))
           :nodes [(->node)]
           :edges {}
           :active (->suffix 0 0 -1)}]
    (reduce add-prefix t (range (count s)))))



''''
(defn st [& xs]
  (reduce (fn [a s]
            (let [word-idx (count (:words a))]
              (as-> a *
                    (assoc *
                      :str s
                      :n       (dec (count s))
                      :active  (->suffix 0 0 -1)
                      :w     word-idx
                      :words (conj (:words * []) s))
                    (reduce add-prefix * (range (count s)))


                                        ; (update-in * [:nw 1] conj word-idx)
                    )))
          {:nodes [(->node)], :edges {}} xs))

; (->suffixtree "cacao")


(defn scontains? [subs idx fulltxt]
  (if (seq subs)
    (if (= (nth fulltxt idx) (first subs))
      (recur (next subs) (inc idx) fulltxt)
      false)
    true))

; (assert (scontains? [2 3] 2 [0 1 2 3 4 5]))


(defn index-of [tree s]
  (let [|s| (count s)
        tree-str (:str tree) tree-edges (:edges tree)]
    (loop [cur_node 0, i 0, ln nil, edge nil]
      (if (< i |s|)
        (when-some [edge (get tree-edges [cur_node (nth s i)])]
          (let [ln (min (inc (edge-length edge)) (- |s| i))]
            (when (scontains? (subvec (vec s) i (+ i ln))
                              (:fci edge) tree-str)
              (recur (:dni edge) (+ i (edge-length edge) 1) ln edge))))
        (+ (:fci edge) (- |s|) ln)))))


'''
(defn find-subs-data [tree s]
  (let [|s| (count s)
        tree-str (:str tree) tree-edges (:edges tree)]
    (loop [cur_node 0, i 0, ln nil, edge nil]
      (if (< i |s|)
        (when-some [edge (get tree-edges [cur_node (nth s i)])]
          (let [ln (min (inc (edge-length edge)) (- |s| i))]
            (when (scontains? (subvec (vec s) i (+ i ln))
                              (:fci edge) tree-str)
              (recur (:dni edge) (+ i (edge-length edge) 1) ln edge))))
        {:index (+ (:fci edge) (- |s|) ln)
         ;; :edge edge
         :node cur_node}))))

(comment

  (time (find-subs (time (->suffixtree (time (slurp "/home/jano/wordlist-hu-0.3/list/freedict"))))
                   "song"))






(assert (nil? (find-subs (->suffixtree "dolorem ipsum dolor sit amet") "ipsumedo")))
(assert (integer? (find-subs (->suffixtree "dolorem ipsum dolor sit amet") "olo")))

(find-subs-data (->suffixtree "dolorem ipsum dolor sit amet") "olo")

;; TODO: massive debug!! pl.: tuple elso fele miert mindig nil??


  )
