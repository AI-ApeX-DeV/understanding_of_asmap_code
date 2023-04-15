# understanding_of_asmap_code
detailed understanding of classes and functions in the asmap code 

ASNEntry: a class that represents an Autonomous System Number (ASN) entry, which is a tuple of two elements. The first element is a list of boolean values representing an IP prefix, and the second element is an integer representing the ASN. The class contains a few helper methods for comparison and hashing.

ASMap: a class that represents a map of IP prefixes to ASNs. The map is implemented as a trie, where each node is a tuple of two elements: a boolean value representing the current bit in the prefix, and either an ASN or another tuple representing a subtree of the trie.
           	
The helper functions in this module are:
 
net_to_prefix: a function that converts an IPv4 or IPv6 network to a prefix represented as a list of bits. If the network is an IPv4 range, it is remapped to its IPv4-mapped IPv6 range (::ffff:0:0/96).

prefix_to_net: the reverse of net_to_prefix, converting a prefix represented as a list of bits to an IPv4 or IPv6 network.

_VarLenCoder: a class representing a custom variable-length binary encoder/decoder for integers. It is used internally by the ASMap  class to encode and decode ASN values efficiently.

net_to_prefix(net: Union[ipaddress.IPv4Network,ipaddress.IPv6Network]) -> List[bool]: This function takes an IPv4 or IPv6 network object and returns its corresponding prefix as a list of boolean values, where each element in the list represents a bit in the prefix.

prefix_to_net(prefix:List[bool])-> Union[ipaddress.IPv4Network,ipaddress.IPv6Network]: This function is the reverse of net_to_prefix, taking a list of boolean values representing a prefix and returning an IPv4 or IPv6 network object.

ASNEntry = Tuple[List[bool], int]: This is a type alias for a tuple that represents an (IP prefix, ASN) entry, where the IP prefix is represented as a list of boolean values and the ASN is an integer.

ASNDiff = Tuple[List[bool], int, int]: This is a type alias for a tuple that represents a (IP prefix, old ASN, new ASN) entry, where the IP prefix is represented as a list of boolean values and the old and new ASNs are integers.

_VarLenCoder: This is a class that represents a custom variable-length binary encoder and decoder for integers. The class has a constructor that takes two parameters: minval (an integer representing the minimum value that can be encoded) and clsbits (a list of integers representing the number of bits used to encode each class of integers). The class has three methods: can_encode checks whether a given integer can be encoded using the current minval and clsbits; encode encodes an integer and appends the result to a list of integers; and encode_size computes the number of bits required to encode an integer.

build_map(asns: List[ASNEntry], full_tree: bool = True) -> List[ASNDiff]: This function takes a list of (prefix, ASN) entries and returns a list of (prefix, old ASN, new ASN) entries that represent changes in the ASN for each prefix. The full_tree parameter is a boolean that determines whether the function should build a full tree of ASNs or only the specified ones.

_build_map_recursive(asns: Dict[int, List[ASNEntry]], start_bit: int, end_bit: int, use_clustering: bool) -> List[ASNDiff]: This is a helper function for build_map that recursively builds a list of (prefix, old ASN, new ASN) entries. It takes a dictionary of (ASN, [(prefix, ASN)]) pairs, a start bit, an end bit, and a boolean that determines whether to use clustering to optimize the prefix list.

build_optimized_map(asns: List[ASNEntry]) -> List[ASNDiff]: This function takes a list of (prefix, ASN) entries and returns an optimized list of (prefix, old ASN, new ASN) entries that represent changes in the ASN for each prefix.

_build_optimized_map_recursive(asns: Dict[int, List[ASNEntry]], start_bit: int, end_bit: int) -> List[ASNDiff]: This is a helper function for build_optimized_map that recursively builds an optimized list of (prefix, old ASN, new ASN) entries. It takes a dictionary of (ASN, [(prefix, ASN)]) pairs, a start bit, and an end bit
_build_clustered_prefix_list(prefixes):This function takes a list of prefixes and builds a prefix list with clusters of consecutive IP prefixes collapsed into a single summary prefix. For example, if the input prefix list contains the following  prefixes:
 
The output prefix list will contain the following clusters:
 
This function is used to simplify the prefix list by collapsing consecutive prefixes into a smaller number of summary prefixes, which reduces the number of rules that need to be installed on the router.


process_rules(rules): This function takes a list of rules and processes them to generate a prefix list that can be installed on the router. It first builds a dictionary of ACL rules based on the IP protocol, and then creates a prefix list for each protocol by merging the ACL rules and building a clustered prefix list.
generate_prefix_lists(acls):This function takes a dictionary of ACLs and generates prefix lists for each ACL using the process_rules() function. The output of this function is a dictionary of prefix lists, where the keys are the names of the ACLs and the values are the prefix lists.

_to_entries_flat(self, fill: bool = False) -> List[ASNEntry]: This function converts an ASMap object to a list of non-overlapping (prefix, asn) tuples. The fill parameter, if True, permits the resulting ASNEntry objects to cover subnets that are unassigned in the ASMap object. This can result in a shorter list.

_to_entries_minimal(self, fill: bool = False) -> List[ASNEntry]: This function converts an ASMap object to a minimal list of ASNEntry objects, exploiting overlap. The fill parameter, if True, permits the resulting ASNEntry objects to cover subnets that are unassigned in the ASMap object. This can result in a shorter list.
str (self) -> str: This function returns a string containing Python code constructing the ASMap object.

to_entries(self, overlapping: bool = True, fill: bool = False) -> List[ASNEntry]: This function converts the mappings in the ASMap object to a list of ASNEntry objects. The overlapping parameter, if True, permits the subnets in the resulting ASNEntry objects to overlap. This can result in a shorter list. The fill parameter, if True, permits the resulting ASNEntry objects to cover subnets that are unassigned in the ASMap object. This can result in a shorter list.

from_random(num_leaves: int = 10, max_asn: int = 6, unassigned_prob: float = 0.5) -> "ASMap": This static method constructs a random ASMap object. The num_leaves parameter specifies the number of leaves in the trie, the max_asn parameter specifies the maximum ASN value, and the unassigned_prob parameter specifies the probability for leaf nodes to be unassigned.

_to_binnode(self, fill: bool = False) -> _BinNode: This function converts a trie to a binary node. The fill parameter, if True, permits the resulting ASNEntry objects to cover subnets that are unassigned in the ASMap object. This can result in a shorter list.

The test_asmap_roundtrips function tests whether random ASMap objects can be round-tripped to/from entries/binary. It generates random ASMap objects with varying numbers of leaves, bits in the AS numbers used, and probability that leaves are unassigned. For each object, it tests the following: 
1. Construct an ASMap object from random parameters.
2. Convert the ASMap object to entries and then back to an ASMap object for overlapping and non- overlapping entries.
3. Assert that the new ASMap object is equal to the original one.
4. Convert the ASMap object to binary and then back to an ASMap object.
5. Assert that the new ASMap object is equal to the original one.

The test_patching function tests the behavior of the update, lookup, extends, and diff methods of the ASMap class. It generates random ASMap objects with varying numbers of leaves, bits in the AS numbers used, and probability that leaves are unassigned. For each object, it makes five patches, each building on top of the previous ones, and checks the following:

1. Whether there is any difference between the original ASMap object and the patched one.
2. Whether the original ASMap object extends the patched one, and the other way around.
3. For every difference found, whether the original ASMap and patched ASMap objects actually differ there, and whether the lookup holds for extended paths.
4. Whether there is a patch for every extended path.
5. The unittest module is used to run the test cases.

_BinNode and ASMap, which are used for mapping IP subnets to ASNs (Autonomous System Numbers). The _BinNode class represents a node in a binary trie used to store the subnet-to-ASN mappings, while the ASMap class represents the entire mapping.

The _BinNode class has several static methods and two constructors. The constructors create a new _BinNode object with an instruction and up to two arguments, where the instruction is an enumeration value of type _Instruction. The    init    constructor accepts three parameters: an instruction, arg1, and arg2. The overload decorator is used here to specify two possible signatures for this method. The first signature expects arg1 to be an integer and arg2 to be a _BinNode object. The second signature accepts None as default values for both arg1 and arg2.

The _BinNode class also provides several static methods, including make_end(), which creates a new _BinNode with an END instruction, make_leaf(val), which creates a new _BinNode with a RETURN instruction and val as its argument, and make_branch(node0, node1), which creates a new _BinNode with a JUMP or MATCH instruction, depending on the types of the two given nodes. The method make_default(val, sub) creates a _BinNode with a DEFAULT or RETURN instruction, depending on the type of the given subnode, and val as its argument.

The ASMap class represents a mapping from IP subnets to ASNs using a binary trie. The class provides several methods, including update(prefix, asn), which adds or updates a mapping from the given prefix to the given ASN in the trie, lookup(ip), which looks up the ASN for the given IP address in the trie, and to_list(), which converts the trie to a list of ASNEntry objects. The ASNEntry class is not defined in this code.

The ASMap class also has several static methods, including from_list(entries), which creates a new ASMap object from the given list of ASNEntry objects, and from_bytes(data), which creates a new ASMap object from the binary asmap file format. The to_bytes() method converts the trie to the binary asmap file format.

The ASMap    class implements the total_ordering decorator to provide comparison methods based on the subnets.

_Instruction(Enum) - An enum that defines the possible instruction types for the binary node.

_Instruction.RETURN, _Instruction.JUMP, _Instruction.MATCH, and _Instruction.DEFAULT are the four possible values.

_BinNode - A class that represents a binary node in the compressed data structure. It contains an instruction type, and arguments depending on the instruction type.

_ASIDEncoder - A class that is responsible for encoding and decoding the AS number.

_JumpEncoder - A class that is responsible for encoding and decoding the offset of the jump.

_MatchEncoder - A class that is responsible for encoding and decoding the match bits.

_InsEncoder - A class that is responsible for encoding and decoding the instruction type.

ASMap - A class that implements the compressed data structure for storing mappings between Autonomous System Numbers (ASNs) and their corresponding IP subnets. It has the following methods:

init (self) - A constructor that initializes the trie to an empty list.

_set_trie(self, trie) - A method that sets the trie to a given trie.

get(self, asn) - A method that returns the IP subnet associated with a given ASN. Returns None if the ASN is not found in the trie.

set(self, asn, subnet) - A method that adds a new mapping between an ASN and its corresponding IP subnet to the trie.

len (self) - A method that returns the number of mappings in the trie.

contains (self, asn) - A method that returns True if the ASN is found in the trie, and False otherwise.

iter (self) - A method that returns an iterator over the ASNs in the trie.

_to_binnode(self, fill) - A method that converts the trie to a binary tree of _BinNodes.

_to_str(self, indent) - A method that returns a string representation of the trie.

str (self) - A method that returns a string representation of the trie.

_merge(self, other) - A method that merges another ASMap object with this one.

_get_subnet(self, sub, prefix) - A method that returns the IP subnet with the given prefix.

_get_first_diff_bit(self, left, right) - A method that returns the position of the first bit where left and right differ.

_split_subnet(self, sub, prefix) - A method that splits an IP subnet into two subnets at the given prefix.

_find_hole(self) - A method that returns the first unused subnet prefix.

_join(self, left, right) - A method that returns a new IP subnet that is the combination of two subnets.

_optimize(self) - A method that optimizes the trie by merging adjacent subnets.

merge(self, other) - A method that merges another ASMap object with this one.

set_prefix(self, asn, prefix) - A method that sets the prefix of the IP subnet associated with a given ASN.

compress(self) - A method that compresses the trie by removing unused subnets.

init (self, items: Iterable[ASMPair] = ()): initializes an empty ASMap object, or one populated with the given list of items, where each item is a pair (tuple) consisting of an AS number and an IP address prefix.
len (self) -> int: returns the number of items in the ASMap.

eq (self, other: Any) -> bool: returns True if the given ASMap object is equal to this ASMap object, i.e., they have the same items.

getitem (self, asn: int) -> List[ASMPair]: returns a list of pairs of AS numbers and IP address prefixes that match the given AS number.

setitem (self, asn: int, prefix: str) -> None: sets the given IP address prefix for the given AS number.

delitem (self, asn: int) -> None: removes the item with the given AS number from the ASMap.

iter (self) -> Iterator[ASMPair]: returns an iterator over the items in the ASMap.

contains (self, item: Union[ASMPair, int]) -> bool: returns True if the given pair (tuple) of AS number and IP address prefix or the given AS number is in the ASMap.

from_random(cls, num_leaves: int, max_asn: int, unassigned_prob: float) -> ASMap: returns a random ASMap object with the given number of leaves, maximum AS number, and unassigned probability.

to_entries(self, overlapping: bool = False, fill: bool = False) -> List[ASMPair]: returns a list of pairs (tuples) of AS numbers and IP address prefixes in the ASMap. If overlapping is True, the returned list may have overlapping prefixes. If fill is True, the returned list may include pairs with unassigned AS numbers.

extends(self, req: "ASMap") -> bool: determines whether this ASMap matches the given ASMap object for all subranges where the given ASMap object is assigned.

diff(self, other: "ASMap") -> List[ASNDiff]: computes the diff from this ASMap object to the given ASMap object.

copy (self) -> "ASMap": constructs a copy of this ASMap object. Its state will not be shared.

deepcopy (self, _) -> "ASMap": constructs a deep copy of this ASMap object. Its state will not be shared.

test_ipv6_prefix_roundtrips(self) -> None: tests that random IPv6 network ranges roundtrip through prefix encoding.

test_ipv4_prefix_roundtrips(self) -> None: tests that random IPv4 network ranges roundtrip through prefix encoding.

test_asmap_roundtrips(self) -> None: tests that random ASMap objects roundtrip to/from entries/binary.


